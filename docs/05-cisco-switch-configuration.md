# Cisco Catalyst Configuration

## Management objective

The Cisco Catalyst was not purchased new or factory-reset. I found a **used Catalyst 2960X-24PS-L for $20 at a local used-computer store** and deliberately incorporated it into the lab as-is.

That changed the task from "configure a blank switch" to something closer to inheriting infrastructure from another environment: **discover what was already configured, regain administrative access, determine whether the existing state was safe/useful, and then establish a known management baseline without blindly erasing evidence.**

The switch already had configuration on it, including the hostname `LaughTale`, a local privileged user entry, and existing interface/configuration state. I therefore treated the device as **unknown inherited infrastructure**, not as a clean appliance.

The goal became:

1. Obtain reliable console access.
2. Inspect the existing configuration before changing it.
3. Determine how the active ports were behaving.
4. Establish a known management IP and default gateway.
5. Re-establish local administrative authentication.
6. Restrict remote CLI management to SSH.
7. Save the known-good configuration.
8. Validate the actual forwarding behavior of the ports used by the lab.

This $20 switch became one of the most useful pieces of the build because its previous life created troubleshooting work that a factory-new switch would not have provided.

## Inherited configuration and port state

Because the switch was second-hand, I did **not** assume that every port was factory default.

Part of the investigation was determining whether Zoro's initial Gi1/0/3 connectivity problem was caused by an old VLAN assignment, shutdown state, speed/duplex setting, or other leftover interface configuration.

I inspected the live switch with commands such as:

```text
show running-config
show startup-config
show ip interface brief
show interface status
show running-config interface gi1/0/2
show running-config interface gi1/0/3
show running-config interface gi1/0/4
show mac address-table dynamic
```

The important finding was that the **final inspected interface stanzas for Gi1/0/2, Gi1/0/3, and Gi1/0/4 contained no special per-interface configuration**. All three were operating on VLAN 1, and the switch ultimately learned the connected hosts dynamically.

That means the switch absolutely **arrived preconfigured**, but the later evidence did not support claiming that a leftover custom configuration on port 3 caused Zoro's transient failure. I kept those two facts separate instead of rewriting the incident after the fact.

This is also why I did not immediately factory-reset the Catalyst. Inspecting inherited state was itself part of the exercise.

## Console access

A Cisco-compatible USB-to-RJ45 console cable enumerated in Windows as a USB serial port. PuTTY was used at the standard Cisco console settings, including 9600 baud.

## Management SVI

The switch management interface was configured on VLAN 1:

```text
interface Vlan1
 ip address 192.168.0.2 255.255.255.0
```

The default gateway was set to:

```text
ip default-gateway 192.168.0.1
```

## SSH

A local privileged administrator was configured and VTY lines were restricted to SSH authentication using the local user database.

Representative sanitized configuration:

```text
username <admin-user> privilege 15 secret <redacted>

ip ssh version 2

line vty 0 15
 login local
 transport input ssh
```

Configuration was persisted with:

```text
write memory
```

### IOS modes learned

- `>`: user EXEC
- `enable`: enter privileged EXEC
- `#`: privileged EXEC
- `configure terminal`: enter global configuration
- `(config)#`: global configuration mode
- `running-config`: active configuration in RAM
- `startup-config`: persisted boot configuration
- `write memory`: save the active configuration

## Legacy SSH compatibility

The older IOS SSH stack did not initially negotiate with a modern Windows OpenSSH client. The client reported incompatible key-exchange, host-key, and MAC algorithms in sequence.

For this lab, compatibility was enabled **client-side for the specific connection** rather than weakening modern defaults globally. The required legacy families included:

- Diffie-Hellman group14 SHA-1 key exchange
- RSA/SHA-1 host key
- HMAC-SHA1

This is documented as a legacy-device interoperability constraint, not a recommended modern cryptographic baseline.

## Why the $20 purchase mattered

From a lab perspective, the inexpensive used switch created a realistic infrastructure-administration scenario:

```text
Used enterprise hardware
        ↓
Unknown prior configuration
        ↓
Need console access
        ↓
Inspect before modifying
        ↓
Establish management baseline
        ↓
Troubleshoot live forwarding
        ↓
Persist known-good state
```

The lesson was not "cheap switch equals easy networking." It was that **used enterprise equipment often comes with history**, and an engineer needs to discover that state rather than assume a blank slate.

## Security note

No switch passwords, secret hashes, or authentication material are stored in this repository.
