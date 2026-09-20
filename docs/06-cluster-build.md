# ThousandSunny Cluster Build

## Objective

Create a three-node Proxmox cluster with deterministic node naming and the desired membership order:

| ID | Node | Link 0 |
|---:|---|---|
| 1 | Luffy | `192.168.0.22` |
| 2 | Zoro | `192.168.0.23` |
| 3 | Sanji | `192.168.0.24` |

## Cluster creation

The cluster was created from Luffy:

**Datacenter → Cluster → Create Cluster**

- Cluster name: `ThousandSunny`
- Link 0: `192.168.0.22`

Luffy became node ID 1.

## Node joins

Joining nodes used Proxmox's generated cluster join information from an existing cluster member. Join tokens/fingerprints are operational data and are intentionally not committed.

A joining node must be treated carefully because joining replaces its local cluster configuration. Fresh nodes without important VMs/containers were used for this build.

## Membership-order correction

Sanji was initially joined before Zoro, becoming node ID 2. Because the desired lab convention was Luffy=1, Zoro=2, Sanji=3, Sanji was removed and cleaned, Zoro was joined, and Sanji was rejoined.

The recovery path exposed the underlying mechanics of Proxmox clustering and is documented separately in [the recovery case study](07-cluster-recovery-case-study.md).

## Final state

```text
ThousandSunny
├── 1  luffy  192.168.0.22
├── 2  zoro   192.168.0.23
└── 3  sanji  192.168.0.24
```

The GUI reported three cluster nodes after the final join.
