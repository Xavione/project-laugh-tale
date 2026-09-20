# Runbook: Quorum Troubleshooting

## What quorum protects

Quorum prevents a minority partition from independently modifying shared cluster state. It is a safety mechanism against split-brain behavior.

## Observe first

```bash
pvecm status
pvecm nodes
```

Look at:

- Nodes
- Expected votes
- Total votes
- Quorum
- Quorate flag
- Membership information

## Questions before intervention

1. Are missing nodes actually powered off?
2. Could there be a network partition?
3. Is another partition of the cluster still operating?
4. Is the Corosync network reachable?
5. Is this planned maintenance or an unexplained failure?

## Lab-specific two-node recovery

During the Sanji removal exercise, Luffy was intentionally the only surviving member. The temporary command used was:

```bash
pvecm expected 1
```

This restored quorum for the controlled membership operation.

**Do not treat this as a generic fix for quorum loss.** Forcing the expected vote count while another partition is alive can defeat the protection quorum is designed to provide.

## Related components

- `pvecm`: Proxmox cluster management CLI
- Corosync: cluster messaging/membership
- votequorum: voting/quorum provider
- `pmxcfs`: Proxmox cluster filesystem backing `/etc/pve`
