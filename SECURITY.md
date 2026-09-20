# Security Policy

## Repository hygiene

This repository documents a real homelab but intentionally excludes authentication material.

Do not commit:

- passwords
- password hashes
- Proxmox cluster join blobs
- Corosync authentication keys
- SSH private keys
- API tokens
- personal email addresses
- session cookies
- screenshots containing credentials

RFC1918 addresses used inside the lab are documented to make topology and troubleshooting understandable.

## Legacy systems

Some lab hardware may require compatibility with older cryptographic algorithms. Any such exception should be narrowly scoped, documented, and treated as technical debt rather than a recommended security baseline.
