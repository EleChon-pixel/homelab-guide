# How I wired it

I run the web apps and their databases in Docker. I use Caddy as the HTTPS entry point for browser traffic, AdGuard Home for private DNS, and Tailscale when I am away from the LAN.

My device looks up a private app name through AdGuard Home, then connects to Caddy over the LAN or Tailscale. Caddy sends the HTTPS request to the app:

```text
name lookup: my device → AdGuard Home → server address
web request: my device → LAN or Tailscale → Caddy → web app
```

Several app containers publish their web ports on the host's loopback interface. Caddy routes private hostnames to Homepage, Uptime Kuma, Grafana, Immich, Navidrome, Jellyfin, Nextcloud, Portainer, Radarr, Prowlarr, Deluge, and the demo site. AdGuard Home resolves those names to the server; I do not need a public DNS record for each app.

The data containers stay behind the apps. Immich uses its own PostgreSQL, Redis, and machine-learning containers. Nextcloud has its own database and Redis. Prometheus reads host and container metrics from Node Exporter and cAdvisor; Grafana displays them. DIUN and the Signal components handle image update notices. The [service map](services.md) shows every container in the snapshot.

## Why the examples look different

My server uses loopback-bound upstreams for several Caddy routes. The small [Compose example](../examples/docker/compose.example.yml) keeps Caddy and two sample apps on one Docker network, so it can use service names as upstreams. It demonstrates the same routing idea without copying my live network settings.

The guide also keeps the app dependencies visible. Recreating a container is easy; recovering the photos, files, databases, and Caddy CA state needs a separate [backup plan](backups.md).
