# Private DNS and networking

A homelab works best when **name resolution**, **network reachability**, and **application routing** are treated as separate jobs.

```text
Device -> resolver -> private name -> reachable Caddy address
Device -> LAN/Tailscale -> Caddy HTTPS -> application
```

## 1. Pick private names

The examples use `home.lab.test` and `status.lab.test`. The `.test` suffix is reserved for examples, so **replace these with names you deliberately configure** in your private DNS environment. Do not paste actual hostnames or tailnet identity into a public repository.

Avoid editing public DNS just to resolve a purely local service. Use a capable resolver such as AdGuard Home or another DNS service, and ensure clients actually use it.

## 2. Choose who can reach what

- **LAN** clients need a route to the reverse proxy on your LAN.
- **Tailscale** clients need tailnet connectivity, an authorized access rule and a route to the service.
- **Docker containers** need shared networks to resolve each other's service names, or an explicit reachable host endpoint.
- A port published as `127.0.0.1:HOSTPORT:CONTAINERPORT` is available **only on the Docker host**, not directly from another LAN device.

## 3. Verify in order

```bash
getent hosts home.lab.test
curl --head https://home.lab.test/
docker compose ps
```

The HTTPS check assumes the device trusts the issuer of the certificate. When `tls internal` is used, Caddy's private CA must be installed as a trusted certificate authority on each client. Treat CA keys as secrets.

If DNS works but HTTPS times out, investigate the listener and firewall. If HTTPS returns a proxy error, verify the Caddy upstream and container network. If only one Tailscale Serve port fails, inspect its exact grant/ACL and Serve route.

## 4. Avoid accidental exposure

Binding to `0.0.0.0` makes an app accessible on host interfaces unless restricted by firewall. Don't copy Compose ports from tutorials without deciding which clients should reach them. Keep administration tools private, separate guest networks from trusted hosts where practical, and maintain an inventory of exposed listeners.

See [remote access](remote-access.md), [reverse proxy](reverse-proxy.md) and [security](security.md).
