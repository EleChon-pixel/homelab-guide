# Keeping the guide safe to share

This repository describes service roles and example wiring. It does not contain the live Caddyfile, DNS rewrites, tailnet identity, IP addresses, volume paths, credentials, certificates, or app data.

The private network is still a network. Portainer, DNS administration, and other management interfaces should have their own access controls. Most app web ports can stay bound to loopback or inside Docker; Caddy is the browser-facing entry point. Deluge and Gluetun should keep their intended VPN boundary when adapted on another host.

Do not mount the Docker socket into Homepage or monitoring containers just to show container status. That socket gives broad control over the host. Use a restricted integration only when there is a concrete need for it.

Before pushing a change:

```bash
git diff --cached
git status --short
```

Check for `.env` files, tokens, passwords, private keys, certificate material, database dumps, logs, real hostnames, and real addresses. `.gitignore` only catches known filenames. If a credential reaches Git history, rotate it; a later deletion does not erase it from the earlier commit.
