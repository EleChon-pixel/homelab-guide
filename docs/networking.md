# Private names and DNS

AdGuard Home resolves the names used by the web apps. They all lead to Caddy; Caddy then decides which container should receive the request. The public DNS system does not need records for these private names.

The examples use `home.lab.test` and `status.lab.test`. They are placeholders, not the names of my server. A client on the LAN and a client on Tailscale may need different reachable addresses for the same service. Check which resolver each client actually uses before adding more DNS rewrites.

The path to a page is:

```text
device → private DNS → Caddy on HTTPS → app
```

If a page stops loading, check in that order:

```bash
getent hosts home.lab.test
curl --head https://home.lab.test/
```

The `curl` check assumes the client trusts Caddy's local certificate authority. If DNS works but HTTPS does not, check reachability of Caddy before changing the app container. If Caddy responds with a gateway error, check its upstream and the app logs.
