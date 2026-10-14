# Backups

The running-container list shows what holds data, but it does not prove that a backup job exists or that a restore has worked. This is the data I would account for before relying on the stack:

| Service | Data to preserve together |
| --- | --- |
| Immich | Original photos, PostgreSQL data or consistent export, and app configuration |
| Nextcloud | User files, database export, and app configuration |
| Jellyfin and Navidrome | Libraries if they are the only copies, plus metadata and settings |
| Caddy | `/data`, including the private CA state |
| AdGuard Home | DNS rewrites and settings |
| Homepage, Uptime Kuma, Grafana, Portainer | Configuration, monitor definitions, dashboards, and management state |
| Radarr, Prowlarr, Deluge | Settings and any data that cannot be recreated |

Use each database's supported export or a coordinated snapshot. Copying a live database file without coordinating writes can produce a backup that will not restore. Keep at least one encrypted copy away from the server, with its recovery key stored separately.

The useful test is a restore on a spare machine or isolated directory: bring back one service, sign in, and open a representative file or record. Record what was missing before assuming the rest of the stack is covered.
