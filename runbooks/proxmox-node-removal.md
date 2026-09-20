# Runbook: Controlled Proxmox Node Removal

> Lab runbook. Node removal affects distributed cluster state. Do not mechanically copy these commands into production.

## Before removal

- Confirm the exact node being removed.
- Migrate or remove workloads as appropriate.
- Confirm whether the node is online or intentionally offline.
- Check quorum before changing membership.

```bash
pvecm nodes
pvecm status
```

## Remove from a surviving member

```bash
pvecm delnode <node-name>
```

Then validate:

```bash
pvecm nodes
pvecm status
```

## If quorum prevents the change

Do **not** immediately force quorum. First understand why votes are missing and rule out another live partition.

In the controlled two-node Laugh Tale recovery, the remaining node was intentionally made the only expected voter:

```bash
pvecm expected 1
```

That action was appropriate only because the other member was deliberately being removed and its state was known.

## Removed-node cleanup

A node that will later rejoin may retain old Corosync/Proxmox cluster state. See the [cluster recovery case study](../docs/07-cluster-recovery-case-study.md) for the exact lab incident and cleanup sequence.
