# Service catalogue

These are examples of roles a homelab may provide. Install only what you need and follow each project's current deployment guide. This repository does not claim that every service is running on a particular host.

| Role | Example | Data to plan for |
| --- | --- | --- |
| Dashboard | Homepage | Configuration and links |
| DNS and filtering | AdGuard Home | DNS rewrites, filters, settings |
| Availability | Uptime Kuma | Monitors and notification settings |
| Metrics | Prometheus, Grafana | Scrape configuration, dashboards, optional history |
| Media playback | Jellyfin | Library mounts and metadata |
| Photos | Immich | Originals, database, upload path |
| Files | Nextcloud | User files, database, configuration |
| Container management | Portainer | Management state and restricted access |
| Media automation | Radarr, Prowlarr, Deluge | Configurations and tightly scoped download paths |

Stateful services such as photo and file libraries need application-consistent backups. Media automation has its own legal and access considerations; use it only for content you are entitled to manage. Management interfaces should never be treated as harmless simply because they are behind a reverse proxy.

Check the [Homepage Docker instructions](https://gethomepage.dev/installation/docker/) and [Uptime Kuma installation guide](https://github.com/louislam/uptime-kuma/wiki/%F0%9F%94%A7-How-to-Install) when adapting the example Compose file; image defaults and configuration requirements can change.
