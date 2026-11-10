# Monitoring: availability, performance and updates

Monitoring answers three different questions. Use distinct tools for each one rather than treating a green Docker status as proof that an app works.

| Question | Tools | Example |
| --- | --- | --- |
| **Is the service reachable?** | Uptime Kuma | HTTPS endpoint or private health check |
| **Is the host healthy?** | Prometheus, Node Exporter, cAdvisor, Grafana | CPU, disk, memory and container restart trends |
| **Is new software available?** | DIUN, optional alert bridge | Updated container image notifications |

## Recommended initial checks

Start with **free disk space**, storage I/O, container restart counts, reverse-proxy health, VPN health if used, and backup-job status. For media automation, monitor the **download client API** and **Jellyfin library scans** in addition to simple container uptime.

```bash
docker compose ps
docker ps --format 'table {{.Names}}\t{{.Status}}'
docker system df
df -h
```

Do not publish metrics endpoints or the Docker socket publicly. Exporters and dashboards can reveal host details and should be restricted to trusted networks.

A healthy container can still have broken DNS, a blocked Tailscale port, or a failed library import. Combine metrics with application-level tests and a regular restore test.

Keep webhook URLs, messaging tokens and notification credentials in private configuration. Never paste production notification payloads or logs into a public guide.
