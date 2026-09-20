# Architecture

## Design intent

Project Laugh Tale is a physical homelab designed as an engineering environment rather than a single-purpose Kubernetes demo. Proxmox provides the virtualization substrate so workloads can be created, destroyed, snapshotted, isolated, and rebuilt without repeatedly reinstalling physical hosts.

The current physical cluster is named **ThousandSunny**. The initial three nodes are **Luffy**, **Zoro**, and **Sanji**.

## Physical topology

```text
Internet
   |
Home Router / Gateway
192.168.0.1
   |
Cisco Catalyst 2960X-24PS-L
Mgmt: 192.168.0.2
   |
   +-- Gi1/0/2 --> Luffy --> 192.168.0.22
   +-- Gi1/0/3 --> Zoro  --> 192.168.0.23
   +-- Gi1/0/4 --> Sanji --> 192.168.0.24
```

The router currently enters the switch on Gi1/0/1.

## Logical layers

```text
Future applications / local AI
             |
Kubernetes + platform services
             |
Linux virtual machines
             |
Proxmox VE / ThousandSunny
             |
Cisco Ethernet switching
             |
Physical mini-PC nodes
```

This layering intentionally separates the physical hardware lifecycle from the future Kubernetes workload lifecycle.

## Addressing

| Device | Address |
|---|---|
| Router / gateway | `192.168.0.1` |
| Cisco switch management | `192.168.0.2` |
| Luffy | `192.168.0.22` |
| Zoro | `192.168.0.23` |
| Sanji | `192.168.0.24` |

Network: `192.168.0.0/24`

## Naming model

The naming is thematic, but the architecture remains explicit and operationally readable:

- **Project Laugh Tale**: overall platform/homelab
- **ThousandSunny**: physical Proxmox cluster
- **Luffy, Zoro, Sanji**: physical Proxmox nodes
- Future Kubernetes and service naming can live above this layer without changing the physical topology.

## Current state

The environment is intentionally simple at this stage: a flat LAN and three Proxmox nodes. VLAN segmentation, Kubernetes networks, observability, automation, and AI workloads are planned as later phases so each layer can be learned and documented independently.
