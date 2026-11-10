# Security and safe publishing

The goal is a **reusable public guide**, not an export of anyone's running home server. No production configurations, real device identities, or live secrets belong here.

## Never commit

- `.env` files, API keys, passwords, access tokens, cookies, credentials and configuration exports
- VPN WireGuard/OpenVPN profiles, private keys, SSH keys, Tailscale state or authentication material
- Certificates and private CA material, recovery codes and database connection strings
- App data, database files, logs, backup archives, torrent files, or downloaded media
- Real LAN, WAN or tailnet IPs, personal hostnames/domains, hardware identifiers or account names
- Screenshots showing admin consoles, addresses, credentials, QR codes or recovery material

Use `.env.example` only for **safe placeholders/defaults**. Prefer reserved example names (like `home.lab.test`) and documentation-only values. A loopback address such as `127.0.0.1` is a safe generic default; do not replace it with a real host address in Git.

## Layered security

1. **Access**: keep application admin UIs off the public internet. Prefer LAN, Tailscale, or an authenticated reverse proxy. Restrict individual ports with Tailscale grants/ACLs and host firewall rules.
2. **HTTPS**: use TLS at Caddy or Tailscale Serve. A private Caddy CA must be trusted on clients; never publish its private key.
3. **Authentication**: require login for Sonarr, Radarr, Jellyfin, Seerr and administrative tools. Use strong unique passwords and non-admin accounts for playback.
4. **Docker**: avoid privileged containers; grant `NET_ADMIN` and `/dev/net/tun` only to a VPN container that needs them. A mounted Docker socket is highly sensitive; avoid giving it to dashboards unless strictly required.
5. **Downloads**: if using Gluetun, keep the VPN firewall/kill switch enabled and verify the torrent client's network namespace. Do not bypass the VPN to fix a DNS error.
6. **Storage**: limit file access by user/group, keep secrets outside repo directories and keep backups encrypted.
7. **Maintenance**: update images and packages, monitor failures, and retain tested backups.

**SSH guidance**: disable password login and root login where practical, use keys, and consider per-key `authorized_keys` restrictions for narrow port-forwarding use cases. For an SSH tunnel, permit only necessary destinations with `PermitOpen`; do not open blanket forwarding because one application needs access.

## Pre-commit checklist

```bash
git status --short
git diff --cached
git diff --cached --name-only
```

Review **contents**, not only file names. Run a vetted secret scanner (for example Gitleaks) before publishing. Inspect image metadata and Git history if you suspect prior exposure.

The root [`.gitignore`](../.gitignore) and [media example ignore file](../examples/media/.gitignore) help avoid accidental additions, but ignore rules do **not** protect already-tracked files or erase past commits.

If a credential ever enters Git history, **revoke/rotate it immediately** and investigate repository history. Deleting it from the latest revision does not make it safe again.

## Before copying deployment snippets

Read official product docs and customize mounts, authentication, firewall and backing storage. Do not deploy sample stacks as internet-facing production configurations without security review.

Only download or host media that you are authorized to use.
