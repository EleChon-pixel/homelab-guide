# Networking and DNS

The example uses `home.lab.test` and `status.lab.test` as documentation names. Replace them with names served by your own private DNS. Keep the DNS resolver, Caddy, and applications on a network that clients can actually reach.

## Request path

1. A trusted device asks the private resolver for `home.lab.test`.
2. The resolver returns the address of the Caddy host reachable by that device.
3. The device connects to Caddy on HTTPS port 443.
4. Caddy routes the request to the Homepage container by its Docker service name.

If both local and tailnet clients use the same name, each group must receive an address it can reach. A tailnet address is useful only while the client is connected to that tailnet. Split DNS or separate names may be simpler than one record for every network.

## DNS example

Create private DNS rewrites like:

```text
home.lab.test   → proxy address reachable by this client
status.lab.test → proxy address reachable by this client
```

AdGuard Home can provide these rewrites, but clients must actually use it as their resolver. Do not use a public DNS record for a private service unless you have intentionally designed that exposure.

## Check each layer

```bash
getent hosts home.lab.test
curl --head https://home.lab.test/
```

The second command succeeds only after the Caddy CA is trusted by the client. For troubleshooting, check DNS resolution, TCP reachability, Caddy logs, and then the upstream container in that order. Do not disable certificate validation to make a permanent setup appear healthy.
