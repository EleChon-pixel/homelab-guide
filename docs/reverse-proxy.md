# Caddy

Caddy is the front door for the web interfaces. The setup uses separate private names for the dashboard, status page, Grafana, Immich, Navidrome, Jellyfin, Nextcloud, Portainer, Radarr, Prowlarr, Deluge, and the demo site. The real names and upstream addresses are not in this repo.

The [example Caddyfile](../examples/caddy/Caddyfile) shows only Homepage and Uptime Kuma. It matches the small [Compose example](../examples/docker/compose.example.yml), where Caddy can reach both by Docker service name. The actual server also uses loopback-bound upstreams. Check which network Caddy is on before copying either pattern.

`tls internal` makes Caddy issue certificates from its own local CA. A client must trust that CA to get a valid HTTPS connection. Keep Caddy's `/data` volume backed up and private; it contains certificate state. Do not copy CA keys or certificates into Git.

To check the sample after copying it to a separate working directory:

```bash
docker compose config
docker compose up -d
docker compose exec caddy caddy validate --config /etc/caddy/Caddyfile
```

Validate an edited Caddyfile before reloading it. For a backend that itself speaks HTTPS, validate its certificate rather than disabling upstream verification.
