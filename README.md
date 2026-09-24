# 🏴‍☠️ Project Laugh Tale

> A production-inspired homelab built to turn hands-on infrastructure work into Platform Engineering, SRE, Kubernetes, automation, observability, and local-AI experience.

![Status](https://img.shields.io/badge/status-active%20build-success)
![Proxmox](https://img.shields.io/badge/virtualization-Proxmox%20VE-orange)
![Network](https://img.shields.io/badge/network-Cisco%20Catalyst-blue)
![Cluster](https://img.shields.io/badge/cluster-3%20nodes-success)
![Docs](https://img.shields.io/badge/docs-runbook%20driven-informational)

## Why this project exists

Project Laugh Tale started with a simple goal: build a small three-node homelab and learn infrastructure by operating it rather than only reading about it.

It has since become the foundation for a larger engineering platform: virtualization, Kubernetes, observability, infrastructure automation, networking, self-hosted services, and eventually a private local voice/AI assistant platform.

The emphasis is not just on reaching a working state. I document architecture decisions, failures, troubleshooting, recovery procedures, and the reasoning behind each layer.

## Current architecture

```text
                         Home LAN / 192.168.0.0/24
                                  |
                           Router 192.168.0.1
                                  |
                    Cisco Catalyst 2960X-24PS-L
                    Management: 192.168.0.2
                      /           |           \
                     /            |            \
            Gi1/0/2          Gi1/0/3          Gi1/0/4
              |                 |                 |
          Luffy              Zoro              Sanji
       192.168.0.22       192.168.0.23       192.168.0.24
        Node ID 1          Node ID 2          Node ID 3
              \                |                /
               \_______________|_______________/
                               |
                  Proxmox Cluster: ThousandSunny
```

| Node | Role | Cluster ID | Management IP | Memory |
|---|---|---:|---|---:|
| **Luffy** | Proxmox node / initial cluster member | 1 | `192.168.0.22` | 16 GB |
| **Zoro** | Proxmox node | 2 | `192.168.0.23` | 16 GB |
| **Sanji** | Proxmox node | 3 | `192.168.0.24` | 16 GB |

All three nodes currently share the same Layer 2 LAN and communicate through the Cisco switch.

## What I have implemented

- Installed Proxmox VE 9.x on three physical mini PCs.
- Designed a predictable static addressing scheme aligned with physical switch ports.
- Integrated a **$20 used Cisco Catalyst 2960X-24PS-L** with inherited configuration, inspected its existing state, established a known management baseline, and configured SSH access.
- Validated Layer 1 through Layer 3 connectivity using interface state, MAC learning, and ICMP.
- Created the **ThousandSunny** Proxmox cluster.
- Worked directly with Proxmox quorum, Corosync membership, `pmxcfs`, and cluster configuration.
- Recovered from a real cluster membership/configuration problem instead of rebuilding the affected host.
- Documented operational procedures as reusable runbooks.
- Preserved the environment as a platform for Kubernetes, observability, automation, and local AI work.

## Troubleshooting is part of the project

The repository intentionally documents the ugly parts of the build, not just the final green dashboards. Three incidents have already become engineering case studies:

| Incident | What failed | What I practiced |
|---|---|---|
| **Zoro network path** | Zoro could not reach the gateway on Gi1/0/3 | Fault-domain isolation, Linux bridge/routing checks, Cisco interface state, MAC learning, ICMP validation |
| **$20 used Cisco switch** | Catalyst arrived from a local used-computer store with inherited configuration; USB-console path then failed and modern OpenSSH rejected its legacy algorithms | Inherited-infrastructure discovery, configuration inspection, serial management, IOS modes, VTY/SSH, cryptographic interoperability |
| **Sanji cluster recovery** | Cluster membership cleanup exposed quorum, CFS locking, stale Corosync state, and authentication remnants | Quorum recovery, `pvecm`, `pmxcfs`, Corosync, systemd, distributed-state cleanup |

These are documented as evidence-driven incident reports. Where a root cause was not proven, I say so rather than retrofitting certainty after the system starts working.

- [Zoro Network Path Incident](docs/09-zoro-network-incident.md)
- [Cisco Console & SSH Bootstrap](docs/10-cisco-console-ssh-bootstrap.md)
- [Sanji / Proxmox Cluster Recovery](docs/07-cluster-recovery-case-study.md)

## Engineering case study: cluster membership recovery

The most valuable part of the build so far was not the happy path.

I initially joined Sanji before Zoro, resulting in:

```text
ID 1  Luffy
ID 2  Sanji
```

I wanted the intended logical order to be:

```text
ID 1  Luffy
ID 2  Zoro
ID 3  Sanji
```

Because Proxmox/Corosync node IDs are assigned through cluster membership rather than edited as cosmetic labels, I removed Sanji, added Zoro, cleaned Sanji's stale cluster state, and then rejoined Sanji.

During the process I encountered:

- loss of quorum in a two-node cluster
- a read-only `/etc/pve` state
- a stale CFS lock on the Corosync configuration
- stale `corosync.conf`
- a remaining Corosync authentication key
- a node that still believed it belonged to the previous cluster state

Rather than reinstalling the host, I diagnosed and recovered the cluster state. The full incident is documented in [Cluster Recovery Case Study](docs/07-cluster-recovery-case-study.md).

## Repository map

| Area | Purpose |
|---|---|
| [Architecture](docs/01-architecture.md) | System design, naming, topology, and design intent |
| [Hardware](docs/02-hardware.md) | Current physical infrastructure |
| [Networking](docs/03-networking.md) | IP plan, Cisco switch, SSH, and validation |
| [Proxmox Installation](docs/04-proxmox-installation.md) | Hypervisor deployment process |
| [Cisco Configuration](docs/05-cisco-switch-configuration.md) | Switch management and remote access |
| [Cluster Build](docs/06-cluster-build.md) | ThousandSunny creation and node membership |
| [Kubernetes VM Topology](docs/11-kubernetes-vm-topology.md) | Proposed six-VM placement, capacity, network, and quorum decision |
| [Recovery Case Study](docs/07-cluster-recovery-case-study.md) | Quorum/CFS/Corosync troubleshooting |
| [Lessons Learned](docs/08-lessons-learned.md) | Engineering takeaways |
| [Zoro Network Incident](docs/09-zoro-network-incident.md) | Layered fault isolation from Proxmox through the Cisco switch |
| [Cisco Console & SSH Bootstrap](docs/10-cisco-console-ssh-bootstrap.md) | USB-console failure, serial recovery, IOS management, and SSH compatibility |
| [Node Join Runbook](runbooks/proxmox-node-join.md) | Repeatable node-add procedure |
| [Node Removal Runbook](runbooks/proxmox-node-removal.md) | Controlled removal procedure |
| [Quorum Recovery](runbooks/quorum-recovery.md) | Lab-specific quorum recovery notes |

## Skills demonstrated

**Virtualization:** Proxmox VE, Linux administration, bridges, node lifecycle  
**Networking:** Cisco IOS, switching, VLAN fundamentals, MAC tables, static addressing, SSH  
**Distributed systems:** quorum, voting, cluster membership, stale state, configuration locking  
**Operations:** troubleshooting, validation, recovery, runbook development  
**Next layers:** Kubernetes, Helm, Terraform/Ansible, Prometheus/Grafana, local AI orchestration

## Roadmap

- [x] Rack/compute acquisition
- [x] Managed switching
- [x] Proxmox deployment
- [x] Three-node Proxmox cluster
- [x] Cluster recovery/runbook documentation
- [x] Kubernetes virtual-machine topology ([proposed design](docs/11-kubernetes-vm-topology.md); VMs not deployed)
- [ ] Kubernetes cluster deployment
- [ ] Infrastructure automation
- [ ] Prometheus + Grafana observability
- [ ] Centralized logging
- [ ] GitOps / CI/CD
- [ ] Segmented lab networking/VLANs
- [ ] Local AI inference node
- [ ] Voice interface and automation layer
- [ ] Failure injection and recovery exercises

## Security

Credentials, root passwords, switch secrets, password hashes, Proxmox join tokens, personal email addresses, and other sensitive values are intentionally excluded or redacted. Private RFC1918 lab addressing is shown only to make the architecture reproducible and understandable.

---

**Project status:** Active. This repository follows the build as it evolves, including the mistakes, recoveries, and architecture changes that turn a pile of mini PCs into an operated platform.
