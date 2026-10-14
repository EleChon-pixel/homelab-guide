# How it fits together

The server runs the web apps and their databases in Docker. Caddy is the single HTTPS entry point for people using a browser. AdGuard Home supplies the private names, and Tailscale makes the server reachable from trusted devices away from the LAN.

There are two kinds of containers here:

- **Web apps** such as Homepage, Immich, Nextcloud, Jellyfin, Navidrome, Grafana, and Uptime Kuma. Caddy gives each one a private hostname.
- **Backing services** such as PostgreSQL, Redis, Prometheus exporters, and the Immich machine-learning worker. They do their work without a public web route.

The server proxies several apps through ports bound to its loopback interface. The small Compose example in this repo puts Caddy and its two sample apps on one Docker network and uses service names as upstreams. That example shows the route clearly; the server uses its own configuration.

## Request path

1. A device connects over the LAN or Tailscale.
2. AdGuard Home resolves a private app name to the Caddy host.
3. Caddy handles HTTPS and forwards the request to the app.
4. The app talks to its own database, cache, or storage as needed.

The [service map](services.md) shows which containers belong together. Keeping those dependencies visible matters most for upgrades and restores: restarting a web app is simple, but losing its database or files is not.
