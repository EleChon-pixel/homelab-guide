# Backup and recovery plan

A running container is not a backup. App data, original files, certificates and databases survive a container recreation **only** if their persistent storage is protected.

## What to back up

| Component | Data to preserve |
| --- | --- |
| Jellyfin | Config, user state, metadata; library files if not reproducible |
| Seerr | Configuration/database (requests, users and settings) |
| Radarr, Sonarr, Prowlarr | App configuration/databases, quality profiles, integrations |
| Deluge | Config, torrent state and unfinished downloads if important |
| Gluetun | Deployment settings; **private VPN profile**, stored only in an encrypted secrets backup |
| Immich | Original media and a consistent PostgreSQL backup; config |
| Nextcloud | Files, app config and consistent database export |
| Caddy | `/data` (TLS/CA material), configuration; encrypt and restrict access |
| Homepage, Uptime Kuma, Grafana, Portainer | Config, monitoring definitions, dashboards and management state |
| AdGuard Home | Settings and local DNS rewrites |

For a completed media library, decide whether the files are irreplaceable. If yes, back them up separately; app configuration alone cannot recreate them.

## Sensible strategy

- Keep at least **three copies** of important data, on **two kinds of storage**, with **one off-site or offline copy**.
- Encrypt off-site backups and store recovery keys separately.
- Use supported database exports or coordinated snapshots; don't blindly copy active database files.
- Keep consistent versions of app config and related backing data when possible.
- Schedule backups *and* monitor whether they succeeded.
- Periodically restore a sample service or file into an **isolated** environment.

Do not include real backups, recovery keys, VPN profiles or certificate keys in this repository. The example Compose files are not backup plans.
