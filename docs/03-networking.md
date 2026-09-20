# Networking

## LAN

The lab currently uses `192.168.0.0/24`.

A `/24` prefix corresponds to `255.255.255.0`. The router at `192.168.0.1` provides the default gateway and currently provides DNS for the Proxmox hosts.

## Port map

| Cisco Port | Device | IP |
|---|---|---|
| Gi1/0/1 | Home router | `192.168.0.1` |
| Gi1/0/2 | Luffy | `192.168.0.22` |
| Gi1/0/3 | Zoro | `192.168.0.23` |
| Gi1/0/4 | Sanji | `192.168.0.24` |

The three server-facing interfaces currently operate in VLAN 1 with no special per-interface configuration.

## Troubleshooting port 3

Zoro initially failed to reach the gateway while connected to Gi1/0/3. The same host and cable worked when moved to the known-good Gi1/0/2 path, narrowing the fault domain toward the switch/link path rather than the Proxmox IP configuration.

Useful validation commands included:

```text
show ip interface brief
show interface status
show interface gi1/0/3 status
show mac address-table dynamic interface gi1/0/3
```

After link cycling/reseating and switch inspection, Gi1/0/3 negotiated at 1 Gbps and learned Zoro's MAC dynamically. ICMP validation subsequently succeeded between the switch and all three nodes, the gateway, and an external IP.

The exact transient root cause was not conclusively proven, so the incident is documented as an isolation and validation exercise rather than attributing a cause without evidence.

## Validation strategy

I validated networking at multiple layers rather than relying only on the web GUI:

1. **Link state**: interface connected/up.
2. **Data link**: switch dynamically learned the endpoint MAC.
3. **Network layer**: host and switch ICMP tests succeeded.
4. **Application layer**: Proxmox HTTPS management was reachable on TCP/8006.

This layered approach made it possible to distinguish host configuration problems from physical/switching problems.
