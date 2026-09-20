# Proxmox VE Installation

## Installation media

Proxmox VE 9.x installation media was written to a 16 GB USB drive using Rufus on Windows. The ISOHybrid image was written in **DD image mode**.

## Installation choices

The graphical installer was used. The graphical and terminal installers produce the same Proxmox platform; graphical installation was selected for convenience during the initial build.

Common settings:

- Country: United States
- Time zone: America/Chicago
- Keyboard: en-us
- Filesystem: ext4
- Gateway: `192.168.0.1`
- DNS: `192.168.0.1`

### Luffy

- FQDN: `luffy.laughtale.home.arpa`
- Address: `192.168.0.22/24`

### Zoro

- FQDN: `zoro.laughtale.home.arpa`
- Address: `192.168.0.23/24`

### Sanji

- FQDN: `sanji.laughtale.home.arpa`
- Address: `192.168.0.24/24`

## Post-install validation

Each host was checked for:

- correct static address
- default route
- gateway reachability
- Internet reachability
- Proxmox HTTPS interface
- Linux bridge state

Proxmox creates a bridge such as `vmbr0` so future virtual machines can attach to the physical network through the host.

One early gateway test on Luffy briefly returned `Destination Host Unreachable`; an immediate retry succeeded. This was treated as transient link/ARP convergence rather than changing configuration without evidence.

## Browser warning

The initial Proxmox web interface uses a self-signed TLS certificate. Browser warnings during the lab bootstrap were expected. Replacing/bootstraping certificates is a later hardening task.
