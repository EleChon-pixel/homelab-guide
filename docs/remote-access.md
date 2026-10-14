# Remote access

Tailscale runs on the host and gives my own devices a private route home. AdGuard Home provides the app names; Caddy serves those apps on HTTPS. This keeps the browser URLs consistent between services without assigning a special external port to each app.

The demo site was a good example of why port ownership matters: an extra Tailscale Serve listener on another HTTPS port did not give reliable client access, while Caddy was already listening on the standard HTTPS port. In this guide, the normal pattern is another Caddy hostname. Do not ask Tailscale Serve and Caddy to own the same listener.

In the sample Compose file, `BIND_ADDRESS` is `127.0.0.1` until you change the local `.env`. That is useful for testing on one machine, but a remote device cannot reach it. If you bind Caddy to a tailnet address, keep that address only in the local `.env` and check that Tailscale is up when Docker starts. Review tailnet access rules and the host firewall before exposing port 443 to more clients.

Portainer and other management pages deserve narrower access than a dashboard or media player. Tailscale membership alone should not be their only security decision.
