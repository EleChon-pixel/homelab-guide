# Remote access with Tailscale

Tailscale is an example way to give trusted devices private reachability to the homelab. Install it on the server and on each client that needs access. Use tailnet access controls to limit which identities can reach the server and its management ports.

For the sample Compose file, `BIND_ADDRESS` defaults to `127.0.0.1`, so a remote client cannot connect. To serve through Tailscale, set it to the server's own tailnet address in the host-local `.env`, then verify that the address exists when Docker starts. Another valid design is a host firewall that permits only the tailnet interface; review that rule before binding Caddy to all interfaces.

Publish only the proxy port required by the design. Do not publish every application's web port. A DNS resolver used by remote clients also needs a deliberate private access path.

## Verify

1. Check that server and client are both connected to the tailnet.
2. Confirm the client resolves the chosen private name to an address reachable through the tailnet.
3. Check HTTPS on the proxy and confirm the client trusts the Caddy CA.
4. Confirm a device outside the allowed group cannot reach management services.

Tailscale Serve is an alternative for a single endpoint. Do not configure it on the same host port that Caddy already owns. Record which component owns each listener before making changes.
