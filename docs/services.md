# Services

I captured this service list on 14 October 2026. Tailscale is included because it is part of the access path, although it runs on the host rather than in the container list. I left out real hostnames, ports, mounts, and credentials.

## Access and administration

| Service | Container | Job |
| --- | --- | --- |
| Caddy | `caddy` | HTTPS entry point and reverse proxy |
| AdGuard Home | `adguardhome` | Private DNS rewrites and filtering |
| Tailscale | Host service | Private access away from home |
| Homepage | `homepage` | Dashboard linking to the other apps |
| Portainer | `portainer` | Container management interface |

## Monitoring and notifications

| Service | Container | Job |
| --- | --- | --- |
| Uptime Kuma | `uptime-kuma` | Checks whether selected sites respond |
| Prometheus | `prometheus` | Stores scraped metrics |
| Grafana | `grafana` | Dashboards for those metrics |
| cAdvisor | `cadvisor` | Container metrics |
| Node Exporter | `node-exporter` | Host metrics |
| DIUN | `diun` | Watches container images for updates |
| DIUN Signal bridge | `diun-signal-bridge` | Forwards update notices toward Signal |
| Signal API | `signal-api` | Messaging endpoint for notifications |

The update-notification containers are separate from Uptime Kuma. A new image being available does not mean a service is down, and neither check replaces a backup.

## Photos and files

| Service | Container | Job |
| --- | --- | --- |
| Immich server | `immich_server` | Photo library and API |
| Immich machine learning | `immich_machine_learning` | Background image analysis |
| Immich PostgreSQL | `immich_postgres` | Photo database |
| Immich Redis | `immich_redis` | Cache and job coordination |
| Nextcloud | `nextcloud` | File sync and sharing |
| Nextcloud database | `nextcloud-db` | Application database |
| Nextcloud Redis | `nextcloud-redis` | Cache and locking support |

Immich's originals and database belong in the same recovery plan. The same is true of Nextcloud's files and database. Backing containers are part of the service even when they have no browser page.

## Media and downloads

| Service | Container | Job |
| --- | --- | --- |
| Jellyfin | `jellyfin` | Video library and playback |
| Navidrome | `navidrome` | Music library and playback |
| Radarr | `radarr` | Movie library automation |
| Prowlarr | `prowlarr` | Indexer management |
| Deluge | `deluge` | Download client |
| Gluetun | `gluetun` | VPN side of the download stack; Deluge's web interface is reached through it in this setup |

## Other web service

An nginx container serves a small demo site. It has its own Caddy route, just like the dashboard. Its personal label and content are left out of this repository.
