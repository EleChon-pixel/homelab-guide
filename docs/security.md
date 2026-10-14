# Security checklist

## Access

- Keep administration and application access on a trusted LAN or tailnet. Do not forward router ports solely to make the examples work.
- Allow only required traffic to Caddy. Restrict SSH and management interfaces separately.
- Use per-user accounts and multi-factor authentication where supported. Do not share an administrator account across services.
- Review Tailscale grants or access controls for who can reach the host and ports.

## Secrets and configuration

- Keep `.env`, tokens, passwords, private keys, Caddy CA material, and application databases outside Git. Use a secrets manager or host-local files with restrictive permissions.
- The `.gitignore` reduces accidental commits but is not a security boundary. Inspect `git diff --cached` before every commit and scan history before sharing the repository.
- If a secret is committed, revoke or rotate it. Removing the line in a later commit does not remove it from Git history.

## Containers

- Pin image versions or digests before relying on an example. Read release notes, update one service at a time, and keep a rollback plan.
- Do not mount the Docker socket into dashboard or application containers for convenience. Socket access is effectively host administration.
- Keep app ports internal to the Compose network where possible. The sample Compose file publishes only Caddy, bound to loopback by default.
- Review volume permissions and avoid privileged containers unless the service truly requires them.

## HTTPS

The sample Caddyfile uses `tls internal`, which issues certificates from a local CA. Distribute the CA certificate to trusted clients through a trusted channel and verify it before installation. Do not use `tls_insecure_skip_verify` for upstreams. If a service already provides HTTPS, validate its certificate or connect over a private Docker network using HTTP where appropriate.
