# My homelab

This repository shows how I put my home server together. Most services run in Docker. AdGuard Home handles private DNS, Caddy gives the web apps their own HTTPS names, and Tailscale lets my devices reach them away from home.

I use real service names so the layout makes sense, but replace the connection details. The examples use made-up domains and stay separate from my live configuration. Real addresses, mounts, credentials, and application data stay out of Git.

## What i have running

- **Access:** Caddy, AdGuard Home, Tailscale, Homepage, Portainer
- **Monitoring and notifications:** Uptime Kuma, Prometheus, Grafana, cAdvisor, Node Exporter, DIUN, a DIUN-to-Signal bridge, Signal API
- **Files and photos:** Nextcloud with its database and Redis; Immich with PostgreSQL, Redis, and its machine-learning service
- **Media:** Jellyfin, Navidrome, Radarr, Prowlarr, Deluge, Gluetun
- **Small web apps:** an nginx-hosted demo site

The [service map](docs/services.md) covers every container in the snapshot used for this guide. It also distinguishes the user-facing apps from their backing services.

## The how

A device reaches the server over the local network or Tailscale. AdGuard Home resolves a private name to the server. Caddy accepts the HTTPS request and sends it to the right app. Most app web interfaces are kept off the general network and reached through Caddy. [Architecture](docs/architecture.md) and [networking](docs/networking.md) cover the details.

I prefer one Caddy route per web app over a collection of URLs with special ports. That also solved a problem with a small demo site: its own port worked locally but was unreliable from another device, so I gave it a Caddy route like the rest of the apps. I keep availability checks, performance metrics, and image update notices as separate parts of the stack.

The configuration in `examples/` starts Caddy, Homepage, and Uptime Kuma to show the routing pattern. The other services are covered in the [service map](docs/services.md). To try the example, copy these three files into a directory outside this repository and adjust the local `.env`:

```bash
mkdir -p ../homelab-example/homepage
cp examples/environment/.env.example ../homelab-example/.env
cp examples/docker/compose.example.yml ../homelab-example/compose.yml
cp examples/caddy/Caddyfile ../homelab-example/Caddyfile
cd ../homelab-example
docker compose config
```

The sample binds Caddy to loopback. Read [remote access](docs/remote-access.md) before changing that address or opening ports.

## Notes by topic

- [Services](docs/services.md): what runs and what depends on what
- [Architecture](docs/architecture.md): how I wired the server and its apps
- [Networking](docs/networking.md): private DNS and name resolution
- [Reverse proxy](docs/reverse-proxy.md): Caddy and local HTTPS
- [Remote access](docs/remote-access.md): Tailscale and port ownership
- [Monitoring](docs/monitoring.md): availability, metrics, and update notices
- [Backups](docs/backups.md): data that needs a recovery plan
- [Security](docs/security.md): what stays out of the repo and off the public network
