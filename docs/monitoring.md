# Monitoring and update notices

There are three separate jobs in this stack:

- **Uptime Kuma** checks whether selected sites respond. It is the quickest way to notice that a page is unreachable.
- **Prometheus** collects metrics from **Node Exporter** for the host and **cAdvisor** for containers. **Grafana** is where those metrics are read.
- **DIUN** watches for container image updates. The **DIUN Signal bridge** and **Signal API** form the notification path for those notices.

These answer different questions. A healthy container can still have a broken DNS or proxy route, and an available update does not mean the current version has failed. For an important app, check both the app's private HTTPS name and the host's capacity.

Start with disk space, container restarts, and whether the services people use are reachable. Keep notification credentials and webhook details in local configuration, not in this guide.
