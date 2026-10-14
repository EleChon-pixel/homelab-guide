# Homelab guide

These are notes on how I run my home server. Most services are Docker containers. AdGuard Home handles private DNS, Caddy gives the web apps their own HTTPS names, and Tailscale lets my devices reach them away from home.

I keep the live settings on the server. The examples here use made-up names and can be tried on another machine. Real addresses, domains, mounts, credentials, and application data stay out of Git.

## What is running

- **Access:** Caddy, AdGuard Home, Tailscale, Homepage, Portainer
- **Monitoring and notifications:** Uptime Kuma, Prometheus, Grafana, cAdvisor, Node Exporter, DIUN, a DIUN-to-Signal bridge, Signal API
- **Files and photos:** Nextcloud with its database and Redis; Immich with PostgreSQL, Redis, and its machine-learning service
- **Media:** Jellyfin, Navidrome, Radarr, Prowlarr, Deluge, Gluetun
- **Small web apps:** an nginx-hosted demo site

The [service map](docs/services.md) covers every container in the snapshot used for this guide. It also distinguishes the user-facing apps from their backing services.

## How the pieces fit

A device reaches the server over the local network or Tailscale. AdGuard Home resolves a private name to the server. Caddy accepts the HTTPS request and sends it to the right app. Most app web interfaces are kept off the general network and reached through Caddy. [Architecture](docs/architecture.md) and [networking](docs/networking.md) cover the details.

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
- [Architecture](docs/architecture.md): traffic flow and the two proxy patterns
- [Networking](docs/networking.md): private DNS and name resolution
- [Reverse proxy](docs/reverse-proxy.md): Caddy and local HTTPS
- [Remote access](docs/remote-access.md): Tailscale and port ownership
- [Monitoring](docs/monitoring.md): availability, metrics, and update notices
- [Backups](docs/backups.md): data that needs a recovery plan
- [Security](docs/security.md): what stays out of the repo and off the public network

## Before committing changes

Keep `.env` files, tokens, passwords, certificates, databases, backups, and media outside this repository. Review `git diff --cached` before every commit. `.gitignore` helps, but it cannot protect a secret that has already entered Git history.
