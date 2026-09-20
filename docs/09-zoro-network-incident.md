# Case Study: Zoro Network Path Incident

## Incident summary

During the initial Proxmox bring-up, Zoro had a valid-looking host configuration but could not reach the default gateway while connected to Cisco port **Gi1/0/3**.

The failure became a useful troubleshooting exercise because the host, cable, switch port, VLAN, and IP configuration were all plausible suspects.

## Symptoms

Zoro was configured as:

- Address: `192.168.0.23/24`
- Gateway: `192.168.0.1`
- Proxmox bridge: `vmbr0`

Initial tests from Zoro returned `Destination Host Unreachable` when attempting to reach both the gateway and another Proxmox node.

Host inspection showed the physical interface as UP/LOWER_UP and attached to `vmbr0`, while the routing table contained the expected connected route and default route.

## Fault isolation

Rather than changing the IP configuration immediately, I moved the **same Zoro host and Ethernet cable** from Gi1/0/3 to the known-working Gi1/0/2 path.

Connectivity immediately worked.

That test was important because it held most variables constant:

```text
Same host
Same OS
Same Proxmox configuration
Same IP settings
Same Ethernet cable
Different switch path
```

This narrowed the investigation toward the switch/link path.

## Switch-side investigation

After console access to the Catalyst was established, I inspected interface and forwarding state with commands including:

```text
show ip interface brief
show interface status
show interface gi1/0/3 status
show running-config interface gi1/0/3
show mac address-table dynamic interface gi1/0/3
```

The final inspection showed:

- Gi1/0/3 connected
- VLAN 1
- full duplex
- 1 Gbps negotiation
- Zoro's MAC dynamically learned on Gi1/0/3
- no special per-interface configuration differentiating ports 2, 3, and 4

Several link up/down events occurred while the connection was being cycled and reseated.

## Validation

The switch successfully pinged:

```text
192.168.0.22  Luffy
192.168.0.23  Zoro
192.168.0.24  Sanji
192.168.0.1   Gateway
8.8.8.8       External reachability test
```

The three Proxmox HTTPS interfaces were then reachable from the management workstation.

## Root cause

The exact original cause was **not conclusively proven**.

That distinction matters. The evidence demonstrated a transient switch/link-path problem and later demonstrated healthy forwarding, but it did not justify inventing a more specific root cause.

## Engineering takeaway

The incident reinforced a repeatable troubleshooting pattern:

1. Verify host interface and bridge state.
2. Verify the routing table.
3. Test a known-good physical/network path while holding other variables constant.
4. Inspect switch interface state.
5. Verify Layer 2 learning through the MAC table.
6. Verify Layer 3 reachability with ICMP.
7. Verify the application only after lower layers pass.

The useful result was not merely that Zoro eventually worked. The useful result was narrowing the fault domain with controlled tests and documenting uncertainty where the evidence stopped.
