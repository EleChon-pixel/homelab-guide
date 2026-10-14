# Monitoring

Start with availability, then add metrics. Monitoring should report a problem to a place you actually check; a dashboard alone does not alert you.

## Availability

Uptime Kuma can check the proxy hostnames and selected internal services. Use an HTTPS check from a client that trusts the Caddy CA. A check from inside the same Docker network answers a different question than a check from a remote client, so use both when remote access matters.

Useful checks include DNS resolution, proxy response, app login page, backup job completion, and free storage. Avoid putting credentials in a check URL.

## Metrics

Prometheus can collect host and container metrics; Grafana can graph capacity and trends. Restrict exporters and dashboards to the private network. Start with disk free space, memory pressure, CPU load, container restarts, and backup age. Set alert thresholds after observing normal behavior.

Keep monitoring data separate from the backup of irreplaceable user data. Decide how much metric history is worth retaining before sizing storage.
