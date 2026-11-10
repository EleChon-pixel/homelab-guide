# Self-Hosted Homelab Guide

A practical, privacy-conscious guide to running services on a Linux home server with **Docker Compose**, **Caddy**, **private DNS**, and **Tailscale**. Start small, understand each network boundary, and add services as needed.

> **This is a reference guide, not a copy of a real server.** All hostnames, ports, and paths in the examples are generic. No live IP addresses, credentials, API keys, VPN profiles, certificates, or application data are included.

## What you can build

| Layer | Example tools | Purpose |
| --- | --- | --- |
| Network and access | Tailscale, AdGuard Home, Caddy | Private remote access, local DNS, HTTPS |
| Dashboard and administration | Homepage, Portainer | Service links and container management |
| Media | Jellyfin, Navidrome | Watch and listen to your own library |
| Media automation | Seerr, Radarr, Sonarr, Prowlarr, Deluge, Gluetun | Requests, organization, downloads and VPN isolation |
| Files and photos | Nextcloud, Immich | Sync, storage, photo backup |
| Monitoring | Uptime Kuma, Prometheus, Grafana, cAdvisor, Node Exporter, DIUN | Availability, metrics, update alerts |

These are **optional building blocks**, not a requirement to run every service.

## Architecture at a glance

```text
          Laptop / phone / TV
                   |
        LAN or Tailscale (private)
                   |
        DNS -> Caddy reverse proxy
                   |
      +------------+-------------+
      |            |             |
   Homepage     Jellyfin     Other apps
                                 |
                          Private Docker networks

Media workflow:
Seerr -> Radarr (movies) / Sonarr (shows)
                 |             |
              Prowlarr (indexers)
                 |
            Deluge via Gluetun VPN
                 |
      Completed-download import
                 |
       Movies / TV folders -> Jellyfin
```

Read [architecture](docs/architecture.md) before choosing ports and networks.

## Prerequisites

- A Linux machine with Docker Engine and the **Docker Compose plugin** installed.
- An administrator who can maintain backups and perform updates.
- Storage for app data and, optionally, media; plan permissions before mounting it.
- A local DNS resolver and a trusted private certificate authority **if** you want local HTTPS names.
- A Tailscale account **if** you want private remote access. No public port forwarding is required for this guide.

Check the basics:

```bash
docker version
docker compose version
docker ps
```

## First working example: dashboard + HTTPS

The small example contains **Caddy, Homepage, and Uptime Kuma**. It is designed for learning; its default binding is **loopback only**, not available to other devices.

```bash
mkdir -p ../homelab-demo/homepage
cp examples/environment/.env.example ../homelab-demo/.env
cp examples/docker/compose.example.yml ../homelab-demo/compose.yml
cp examples/caddy/Caddyfile ../homelab-demo/Caddyfile
cd ../homelab-demo
# Review and customize .env locally. Do not commit it.
docker compose config --quiet
docker compose up -d
docker compose ps
```

Private names in the example use the reserved `.test` suffix. To access them, configure your own DNS and trust Caddy's internal CA on your device; see [DNS](docs/networking.md) and [HTTPS](docs/reverse-proxy.md). Never publish your private CA key.

## Add media automation (optional)

The separate [media Compose example](examples/media/compose.example.yml) demonstrates shared paths and the Deluge/Gluetun network boundary. **Do not launch it until you have configured your own private WireGuard profile outside Git.**

Follow the [media automation guide](docs/media-automation.md) in order:

1. Create shared download, movie and TV folders.
2. Start the container stack and configure application login.
3. Connect Prowlarr, Radarr, Sonarr, Deluge and Seerr.
4. Verify completed-download imports and Jellyfin library scans.

Use the services only for content you are authorized to access and download.

## Documentation

| Guide | Learn |
| --- | --- |
| [Services](docs/services.md) | What each component does and what is optional |
| [Architecture](docs/architecture.md) | Traffic flows, storage, and container boundaries |
| [Networking](docs/networking.md) | Private DNS, address resolution and troubleshooting |
| [Reverse proxy](docs/reverse-proxy.md) | Caddy, HTTPS and certificate trust |
| [Remote access](docs/remote-access.md) | Tailscale, ACLs, Serve and firewall scope |
| [Media automation](docs/media-automation.md) | Jellyfin, Sonarr, Radarr, Seerr, Deluge and Gluetun |
| [Troubleshooting](docs/troubleshooting.md) | Step-by-step diagnostic commands |
| [Monitoring](docs/monitoring.md) | Health, metrics and notifications |
| [Backups](docs/backups.md) | What to back up and how to test restores |
| [Security](docs/security.md) | Secrets hygiene, least privilege and safe publishing |

## Safety and scope

Examples deliberately avoid personal domains, LAN/tailnet IPs, credentials, tokens, API keys and private WireGuard material. Copy examples to a separate working directory and keep your real `.env`, SSH keys, TLS material, backups and runtime volumes **out of version control**. A `.gitignore` is only a guardrail; always review staged changes and use secret scanning before pushing.

Contributions that improve correctness, portability and security are welcome. Please avoid posting sensitive deployment logs or real credentials in issues.
