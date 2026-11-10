# Service catalog

This is a menu of self-hosted components, **not** an inventory of a particular home server. Choose the components you need. Run each service with persistent storage only where necessary, and avoid exposing its admin interface publicly.

## Access and administration

| Service | Role | Notes |
| --- | --- | --- |
| Caddy | HTTPS reverse proxy | Frontend for trusted internal names |
| AdGuard Home | Private DNS | Optional; any capable DNS resolver works |
| Tailscale | Private remote network | Usually runs on the host, not as a media container |
| Homepage | Dashboard | Service links and optional status integrations |
| Portainer | Docker administration | Restrict access to administrators |

## Media and downloads

| Service | Role | Dependency |
| --- | --- | --- |
| Jellyfin | Movies and TV playback | Readable library folders |
| Navidrome | Music streaming | Music library |
| Seerr | Requests and approvals | Radarr for movies; Sonarr for TV |
| Radarr | Movie monitoring, importing and naming | Indexer access and download client |
| Sonarr | TV monitoring, importing and naming | Indexer access and download client |
| Prowlarr | Indexer configuration and synchronization | Connects to Radarr and Sonarr via API keys |
| Deluge | Torrent download client | Completed-download paths shared with importers |
| Gluetun | VPN network boundary for Deluge | Private provider configuration and kill switch |

Seerr does **not** download files itself. A request must reach Radarr or Sonarr; the automation application must find a suitable release, send it to the client and import it after completion. [Full walkthrough](media-automation.md).

## Files and photos

| Service | Role | Data considerations |
| --- | --- | --- |
| Nextcloud | File sync and sharing | App config, files, database, Redis |
| Immich | Photo management | Originals, PostgreSQL, Redis, machine learning |

## Monitoring and notifications

| Service | Role |
| --- | --- |
| Uptime Kuma | Availability probes |
| Prometheus | Metrics collection |
| Grafana | Metrics visualization |
| Node Exporter | Host metrics |
| cAdvisor | Container metrics |
| DIUN | Image update notifications |
| Messaging bridge/API | Optional alert delivery integration |

A backing database, cache, or metrics exporter does not need its own public URL. Protect these internal components and back up their state with the parent application.

See [architecture](architecture.md), [backups](backups.md), and [security](security.md) for deployment guidance.
