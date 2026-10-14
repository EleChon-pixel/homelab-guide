# Architecture

## Components

| Layer | Example | Responsibility |
| --- | --- | --- |
| Host | Linux server | Runs containers and stores persistent data |
| Application runtime | Docker Compose | Declares containers, networks, and volumes |
| Private connectivity | Local network and Tailscale | Limits which devices can reach services |
| Name resolution | AdGuard Home or another private resolver | Maps friendly names to the proxy |
| HTTPS entry point | Caddy | Terminates TLS and routes by hostname |
| Availability | Uptime Kuma | Checks whether a service responds |
| Metrics | Prometheus and Grafana | Collects and presents capacity trends |
| Recovery | Encrypted backup on separate storage | Restores configuration and user data |

Clients resolve a private hostname, connect to the Caddy host, then Caddy forwards the request to a service on a Docker network. Applications should not publish their own ports unless there is a specific reason. Management interfaces need their own access rules.

## Build order

1. Prepare the host, updates, storage, and a non-admin account for routine work.
2. Establish private connectivity and DNS.
3. Deploy Caddy and one low-risk app; check HTTPS and name resolution from a client.
4. Add monitoring, backups, and a restore exercise.
5. Add stateful apps one at a time. Record their data paths, database dependencies, and upgrade procedure.

## State and ownership

Compose files are a description of containers, not a backup of their data. Keep application state in named volumes or explicit host directories and document where each lives. Databases need an application-consistent export or snapshot. Document the owner and permissions of every data directory before changing containers.
