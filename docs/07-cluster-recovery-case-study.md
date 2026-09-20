# Case Study: Proxmox Cluster Membership Recovery

## Incident summary

During the initial ThousandSunny build, Sanji joined the cluster before Zoro. This was operationally valid, but it assigned Sanji node ID 2. For this lab I intentionally wanted the physical/logical convention:

```text
Luffy = 1
Zoro  = 2
Sanji = 3
```

Instead of reinstalling hosts, I used the situation to learn how Proxmox cluster membership actually works.

## Initial state

```text
ThousandSunny
├── ID 1: Luffy
└── ID 2: Sanji
```

The plan was:

1. Remove Sanji.
2. Join Zoro so it receives ID 2.
3. Clean Sanji's old cluster state.
4. Rejoin Sanji so it receives ID 3.
5. Validate membership and quorum.

## Failure 1: node removal could not write Corosync configuration

From Luffy:

```bash
pvecm delnode sanji
```

The command attempted to remove node 2 but failed to write the new Corosync configuration:

```text
unable to write '/etc/pve/corosync.conf...' - Operation not permitted
```

### Diagnosis

This was a two-node cluster. With only one voting member available, Luffy did not have normal quorum for cluster configuration changes.

### Recovery

In this controlled lab scenario, where I knew the other member was intentionally being removed, I temporarily changed expected votes:

```bash
pvecm expected 1
```

Then:

```bash
pvecm status
```

showed the remaining node as quorate.

> **Production warning:** changing expected votes is a recovery operation, not a routine way to bypass quorum. In a real outage, first determine whether another partition of the cluster is still alive to avoid split brain.

## Failure 2: CFS lock timeout

A later removal attempt repeatedly returned:

```text
trying to acquire cfs lock 'file-corosync_conf' ...
cfs-lock 'file-corosync_conf' error: got lock request timeout
```

Restarting the Proxmox cluster filesystem restored the service, but the Corosync configuration lock remained stale.

After confirming the controlled state, the stale lock was removed:

```bash
rm -f /etc/pve/priv/lock/file-corosync_conf
```

Then:

```bash
pvecm delnode sanji
```

completed the membership change. A `CS_ERR_NOT_EXIST` message indicated that Sanji was no longer an active Corosync member when the removal attempted to kill it; the configuration removal then proceeded.

> **Production warning:** manually deleting a cluster lock should only be done after understanding why it exists and verifying that no legitimate writer still owns the operation.

## Understanding the components

### `pvecm`

`pvecm` is the Proxmox VE cluster management CLI.

Commands used:

```bash
pvecm nodes
pvecm status
pvecm expected 1
pvecm delnode sanji
```

It exposes cluster membership and quorum operations built on the underlying Corosync cluster.

### Corosync

Corosync provides cluster communication, membership, and the votequorum mechanism used by Proxmox.

A useful mental model:

```text
Proxmox cluster management
          |
        pvecm
          |
      Corosync
          |
membership + messaging + quorum
```

### `pmxcfs`

`pmxcfs` is the Proxmox cluster filesystem. It presents cluster configuration through `/etc/pve`.

That is why `/etc/pve` cannot always be treated like an ordinary local directory. Cluster state, quorum, and locking affect whether configuration can be modified.

## Cleaning the removed node

Removing Sanji from Luffy changed the surviving cluster's view, but Sanji still contained its **old local cluster state**.

On Sanji:

```bash
systemctl stop pve-cluster corosync
pmxcfs -l
```

`pmxcfs -l` starts the Proxmox cluster filesystem in **local mode**, allowing local cluster configuration to be modified without normal Corosync participation.

Sanji reported that it was forcing local mode even though `corosync.conf` existed. That was expected and confirmed the stale cluster configuration was still present.

The stale configuration was removed:

```bash
rm /etc/pve/corosync.conf
killall pmxcfs
systemctl start pve-cluster
systemctl status pve-cluster --no-pager
```

The service returned to `active (running)`.

## Failure 3: stale Corosync identity

The first attempt to rejoin Sanji failed with errors indicating:

```text
authentication key /etc/corosync/authkey already exists
corosync is already running
```

The old cluster configuration was gone, but Corosync still retained authentication state.

I stopped Corosync and removed the stale authentication key:

```bash
systemctl stop corosync
rm -f /etc/corosync/authkey
```

After that cleanup, Sanji could join the current cluster normally through the Proxmox GUI.

## Final state

```text
ThousandSunny
├── ID 1  Luffy   192.168.0.22
├── ID 2  Zoro    192.168.0.23
└── ID 3  Sanji   192.168.0.24
```

The cluster GUI reported **Number of Nodes: 3**.

## What this incident taught me

This exercise made several distributed-systems concepts concrete:

**Membership is state.** Removing a node from one side does not automatically erase the removed node's local view.

**Quorum protects writes.** Losing voting majority can intentionally make cluster configuration unavailable.

**`/etc/pve` is not an ordinary directory.** It is backed by `pmxcfs`, which is cluster-aware.

**Locks matter.** Configuration locks prevent simultaneous writers, and stale locks require careful diagnosis rather than blind deletion.

**Corosync is below the Proxmox GUI.** The GUI makes clustering approachable, but membership, authentication, voting, and communication still exist underneath it.

**Recovery beats reinstalling when the goal is learning.** Reinstalling Sanji would have been faster. Recovering it exposed the actual architecture.
