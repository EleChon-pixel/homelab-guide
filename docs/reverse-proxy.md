# Reverse proxy with Caddy

Caddy is the HTTPS entry point. It chooses an upstream by hostname and keeps application ports off the client-facing network.

The [example Caddyfile](../examples/caddy/Caddyfile) routes `home.lab.test` to `homepage:3000` and `status.lab.test` to `uptime-kuma:3001`. Those upstream names work when all three containers share the Compose network. Replace the hostnames and upstreams for your deployment.

The example uses Caddy's internal certificate authority. Clients need to trust its root certificate to validate HTTPS. Back up the Caddy `/data` volume because it contains certificate state. Keep the CA key private and never commit it. A different certificate strategy is needed if you use a public domain and want publicly trusted certificates.

## Validation

From the working Compose directory:

```bash
docker compose --env-file .env -f compose.yml config
docker compose --env-file .env -f compose.yml up -d
docker compose --env-file .env -f compose.yml exec caddy caddy validate --config /etc/caddy/Caddyfile
```

After editing the Caddyfile, validate before reloading:

```bash
docker compose --env-file .env -f compose.yml exec caddy caddy reload --config /etc/caddy/Caddyfile
```

Test from a client with trusted DNS and CA. If a request fails, inspect Caddy logs and confirm the upstream container is healthy. Keep application-specific authentication enabled even when Caddy is reachable only privately.

For current syntax and certificate behavior, refer to the [Caddyfile guide](https://caddyserver.com/docs/caddyfile/concepts) and [Caddy's service guidance](https://caddyserver.com/docs/running).
