# Hardware Inventory

## Compute

The initial ThousandSunny cluster consists of three mini PCs, each currently equipped with **16 GB RAM**.

### Luffy

- Dell OptiPlex 7090
- Intel Core i5-10400
- 6 cores / 12 threads
- 16 GB DDR4
- Samsung NVMe storage
- Proxmox management IP: `192.168.0.22`

### Zoro

- HP mini-PC platform
- 16 GB RAM
- Samsung PM981a-class 256 GB NVMe observed during installation
- Proxmox management IP: `192.168.0.23`

### Sanji

- HP ProDesk mini-PC platform
- 16 GB RAM
- Proxmox management IP: `192.168.0.24`

## Network

- Cisco Catalyst 2960X-24PS-L managed switch
- 24 access ports
- PoE+ capable
- Four SFP uplinks
- Cisco IOS 15.x generation software
- Dedicated console access via USB-to-RJ45 Cisco-compatible console cable

## Supporting equipment

- UPS-backed power
- 16 GB USB installation media
- Rack hardware and structured cabling are being added incrementally
- Local workstation used for browser management, SSH, imaging, and documentation

## Capacity philosophy

The first objective is not maximum hardware density. It is to create enough physical separation to learn clustering, networking, failure domains, virtualization, orchestration, and recovery. Capacity will be expanded when workloads demonstrate a real need.
