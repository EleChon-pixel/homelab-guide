# HTTPS reverse proxy with Caddy

A reverse proxy provides one controlled HTTPS entry point for multiple internal services. The [minimal Caddyfile](../examples/caddy/Caddyfile) routes example private names to Homepage and Uptime Kuma, as defined in the [basic Compose file](../examples/docker/compose.example.yml).

```caddyfile
{$DASHBOARD_HOST} {
    tls internal
    reverse_proxy homepage:3000
}
```

## How to start

1. Copy the example Caddyfile and Compose file into a **private working directory**.
2. Customize the local `.env` with your intended names and bind address. The default `127.0.0.1` is for on-host testing.
3. Ensure the client can resolve the chosen names to a reachable proxy address.
4. Start the stack and trust Caddy's internal CA on each client that will browse the service.

```bash
docker compose config --quiet
docker compose up -d
docker compose exec caddy caddy validate --config /etc/caddy/Caddyfile
docker compose ps
```

Caddy's `tls internal` uses a private certificate authority. Back up Caddy's `/data` volume securely; it includes sensitive CA material. Never commit real Caddy certificates, CA private keys, or an exported live Caddyfile.

## Common problems

- **DNS resolves but the page times out:** Check the proxy bind address, host firewall, client route and Tailscale policy.
- **Bad Gateway:** Caddy can't reach its upstream; inspect Docker networks and the upstream port.
- **Certificate error:** Client may not trust the private CA, or the requested hostname doesn't match.
- **Port conflict:** Only one process can own a host listener. Check whether Caddy or Tailscale Serve already uses it.

Do not bypass TLS verification as a permanent fix. For local-only services, keep upstreams on a private Docker network or bound to loopback. For VPN/torrent-related services, preserve the Gluetun network boundary described in [media automation](media-automation.md).
