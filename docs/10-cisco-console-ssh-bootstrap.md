# Case Study: Cisco Console and SSH Bootstrap

## Objective

Bring a used **Cisco Catalyst 2960X-24PS-L** under known management control and integrate it into Project Laugh Tale without blindly replacing working IOS software.

This became a multi-stage troubleshooting exercise involving physical console access, Windows device enumeration, Cisco IOS configuration modes, management addressing, and legacy SSH interoperability.

## Stage 1: mini-USB console attempt

The first plan was to use the switch's USB console interface from Windows.

That path failed before PuTTY configuration became relevant: Windows did not enumerate a Cisco USB console device. Device Manager showed ordinary COM/Bluetooth ports, but no usable Cisco USB console interface.

A compatible Cisco USB console driver was not readily available through the current product-download path.

### Decision

Rather than installing an untrusted driver from an arbitrary third-party source or changing IOS, I changed the physical management method.

## Stage 2: dedicated console cable

I obtained a Cisco-compatible **USB-A to RJ45 serial console cable**.

Windows enumerated it successfully as a USB serial port. PuTTY could then connect using the standard Cisco console speed:

```text
9600 baud
8 data bits
no parity
1 stop bit
no flow control
```

The switch presented its existing hostname:

```text
LaughTale>
```

This established reliable out-of-band-style local management before modifying network access.

## Stage 3: understanding Cisco IOS modes

The console session provided hands-on practice with the IOS command hierarchy:

```text
LaughTale>                 User EXEC
enable
LaughTale#                 Privileged EXEC
configure terminal
LaughTale(config)#         Global configuration
```

I also learned the operational distinction between:

- **running-config**: configuration currently active in RAM
- **startup-config**: configuration persisted for the next boot
- **write memory**: save the active configuration to persistent startup configuration

## Stage 4: management addressing

The VLAN 1 switched virtual interface was assigned:

```text
interface Vlan1
 ip address 192.168.0.2 255.255.255.0
```

The Layer 3 default gateway for switch management was:

```text
ip default-gateway 192.168.0.1
```

This made the Catalyst a managed device on the same lab management LAN as the Proxmox hosts.

## Stage 5: VTY and SSH

The local administrator and virtual terminal lines were configured using sanitized values:

```text
username <admin-user> privilege 15 secret <redacted>

ip ssh version 2

line vty 0 15
 login local
 transport input ssh
```

### What VTY means

A console line represents the physical console connection. **VTY lines are virtual terminal lines** used for remote CLI sessions such as SSH.

`login local` tells IOS to authenticate those sessions against the switch's local user database.

`transport input ssh` restricts those VTY sessions to SSH rather than permitting insecure remote protocols such as Telnet.

## Stage 6: modern OpenSSH versus legacy IOS

The first Windows SSH attempt failed even though IP connectivity was working.

The errors appeared sequentially:

1. no matching key-exchange method
2. no matching host-key type
3. no matching MAC

The older IOS SSH implementation offered algorithms disabled by default in current OpenSSH clients.

For this specific lab connection, compatibility was enabled on the **client invocation only**, including the legacy algorithm families:

```text
KEX:      diffie-hellman-group14-sha1
Host key: ssh-rsa
SSH MAC:  hmac-sha1
```

### Terminology

**KEX** means key exchange. It is the cryptographic negotiation used to establish shared session secrets.

**SSH MAC** means Message Authentication Code. It protects message integrity and is unrelated to an Ethernet MAC address.

**ssh-rsa** in this context is an older SSH host-key/signature compatibility requirement.

The switch itself remained configured for SSH version 2.

## Stage 7: validation

After local authentication and VTY configuration were corrected, Windows successfully established an SSH session.

Switch-side validation included:

```text
show ssh
show ip interface brief
show interface status
show mac address-table dynamic
show running-config
show startup-config
```

The active SSH session reported SSH version 2.

The configuration was persisted with:

```text
write memory
```

## Why this incident matters

What began as “plug into the switch” crossed several infrastructure boundaries:

```text
Windows device enumeration
        ↓
serial console transport
        ↓
Cisco IOS privilege/configuration modes
        ↓
Layer 3 management addressing
        ↓
VTY authentication
        ↓
SSH cryptographic negotiation
        ↓
remote network management
```

The failure path was more educational than a plug-and-play switch would have been. It required identifying which layer was actually failing instead of treating every connection problem as one generic networking issue.

## Security note

Passwords, IOS secret hashes, authentication material, and other credentials observed during the build are intentionally excluded from this repository. Legacy SSH algorithms are documented as a compatibility constraint, not a recommended security baseline.
