# Security Policy

## Supported versions
We aim to keep the current release line secure. Older tags are **best effort**.

## Reporting a vulnerability
Please email **indiasales@postgresfirst.com** with details and a proof-of-concept if possible.
We will acknowledge receipt within 72 hours and coordinate a fix & disclosure timeline.

## Hardening notes
- The installer only sets `listen_addresses='*'` if you pass `--allow`.
- You are responsible for firewall rules, network ACLs, and TLS.
- Consider using `pg_hba.conf` with CIDR restrictions and strong passwords.
