# Kubernetes VM topology decision

**Status:** Proposed design, September 24, 2026. No Kubernetes VMs or cluster are claimed to exist yet.

## Decision

Run six Linux VMs across the three existing ThousandSunny Proxmox hosts: one Kubernetes control-plane VM and one worker VM per host. Spread both roles across Luffy, Zoro, and Sanji so loss of one physical host leaves two control-plane members and two workers. Keep the physical Proxmox cluster and the future Kubernetes cluster as separate operational layers.

| Proxmox host | VM | Kubernetes role | vCPU | RAM | Virtual disk | Planned LAN IP |
|---|---|---|---:|---:|---:|---|
| Luffy (`192.168.0.22`) | `k8s-cp-01` | Control plane / etcd member | 2 | 3 GiB | 40 GiB | `192.168.0.31` |
| Luffy | `k8s-worker-01` | Application and platform workloads | 2 | 5 GiB | 60 GiB | `192.168.0.34` |
| Zoro (`192.168.0.23`) | `k8s-cp-02` | Control plane / etcd member | 2 | 3 GiB | 40 GiB | `192.168.0.32` |
| Zoro | `k8s-worker-02` | Application and platform workloads | 2 | 5 GiB | 60 GiB | `192.168.0.35` |
| Sanji (`192.168.0.24`) | `k8s-cp-03` | Control plane / etcd member | 2 | 3 GiB | 40 GiB | `192.168.0.33` |
| Sanji | `k8s-worker-03` | Application and platform workloads | 2 | 5 GiB | 60 GiB | `192.168.0.36` |

**Per physical host:** 4 allocated vCPU, 8 GiB VM RAM, and 100 GiB virtual disk. The other 8 GiB of installed RAM is left uncommitted for Proxmox, filesystem cache, and future small services. Across the cluster, the design allocates 12 vCPU, 24 GiB VM RAM, and 300 GiB of virtual disks. vCPU counts are allocations, not claims about the undocumented CPUs in Zoro and Sanji.

## Placement and networking assumptions

- Existing router `192.168.0.1`, Cisco switch `192.168.0.2`, and Proxmox hosts `.22`–`.24` remain on the current `192.168.0.0/24` LAN. The six VM addresses above are **proposals**: verify router DHCP scope and existing leases before reserving them.
- Attach each VM to its host's LAN-connected Proxmox bridge; use the existing access-port/VLAN 1 arrangement. No VLAN, trunk, second NIC, or 10 Gb link is assumed. Router-provided DNS and gateway are the current starting assumptions.
- Plan separate Kubernetes pod and service address spaces that do not overlap the LAN or each other; for example, `10.42.0.0/16` and `10.43.0.0/16`, subject to checking the eventual Kubernetes distribution/CNI and any VPN routes. These are **design candidates**, not configured networks.
- A highly available API endpoint will need a stable address or name and a chosen failover mechanism. Reserve that decision for deployment design; do not imply the three control-plane VMs alone provide an automatically floating API address.
- Physical switch, router, power, and current shared LAN remain common failure domains. VM disks are assumed local to each Proxmox host; no shared storage, Proxmox live migration, or automatic VM failover is assumed.

## Quorum and capacity boundaries

Three distinct control-plane hosts can form a three-member etcd quorum: two members are needed to write cluster state. Likewise, the existing three-vote Proxmox cluster needs two online hosts for quorum. Losing one host can preserve both quorums **if** the Kubernetes distribution is deployed with three voting members and a usable API endpoint; it removes one worker and its local workloads. This is failure tolerance for one host, not a promise that every application stays available. Losing two hosts removes both quorums.

The 16 GiB host limit is the binding constraint. Start with fixed VM RAM and avoid memory overcommit. The 5 GiB workers are for lightweight services, automation experiments, and modest observability; set Kubernetes workload requests/limits after deployment and measure actual use before adding heavier logging or databases. The 100 GiB virtual disk per host is a ceiling to validate against **actual free storage** and Proxmox disk format: Zoro's documented NVMe is 256 GB, while Luffy's and Sanji's usable storage sizes are not established in the inventory. Thin provisioning does not create physical capacity. Before VM creation, verify free space, snapshots/backups, and CPU capabilities on each host; shrink worker disks or revisit placement if any host cannot safely fit this plan.

This topology makes physical failure domains, etcd quorum, capacity accounting, and service scheduling visible to a Platform Engineering/SRE reviewer. Later local-AI inference can run on separately sized compute or a dedicated node; its model memory, GPU, and storage needs should be measured rather than charged against these small control-plane VMs. The Kubernetes workers can eventually host orchestration and voice-facing services if measured capacity permits.
