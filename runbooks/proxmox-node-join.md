# Runbook: Join a Proxmox Node

> Intended for the Project Laugh Tale lab. Validate against current Proxmox documentation before applying to production systems.

## Preconditions

- Joining host is fresh or contains no workloads that must be preserved.
- Unique hostname and static IP are configured.
- Nodes can reach each other over the intended cluster network.
- Time/DNS/hostname resolution are sane.
- Existing cluster is healthy and quorate.

## Procedure

1. On an existing member, open **Datacenter → Cluster → Join Information**.
2. Copy the generated join information.
3. On the new node, open **Datacenter → Cluster → Join Cluster**.
4. Paste the join information.
5. Verify the peer address/fingerprint and cluster network link.
6. Authenticate using the required cluster-node credentials.
7. Join.
8. Expect the joining node's GUI/session to reconnect as cluster configuration changes.

## Validation

From an existing member:

```bash
pvecm nodes
pvecm status
```

Verify the new node appears with the intended cluster link address and that the cluster is quorate.

## Never commit

- join information blobs
- root passwords
- authentication keys
- private keys
- session tokens
