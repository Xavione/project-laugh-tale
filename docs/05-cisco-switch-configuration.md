# Cisco Catalyst Configuration

## Management objective

The used Cisco Catalyst arrived with existing configuration. The goal was to establish known management addressing and authenticated SSH access without unnecessarily replacing IOS during the initial build.

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

## Security note

No switch passwords, secret hashes, or authentication material are stored in this repository.
