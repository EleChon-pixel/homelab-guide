# Troubleshooting checklist

Work from the **first broken link** in the chain, not the application showing the final symptom. Do not delete volumes, turn off the VPN kill switch, or move seeding torrents as a first step.

## General diagnostics (read-only)

```bash
docker compose ps
docker ps --format 'table {{.Names}}\t{{.Status}}'
docker logs --tail 60 <container-name>
docker inspect <container-name> --format '{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}'
docker inspect <container-name> --format '{{range $n, $_ := .NetworkSettings.Networks}}{{println $n}}{{end}}'
```

Inspect logs before sharing them; they can reveal tokens, private addresses or media titles.

## Problem-to-check map

| Symptom | First check | Typical next action |
| --- | --- | --- |
| Seerr says requested but nothing in Radarr | Request approval/status; default Radarr server; Seerr logs | Test Seerr → Radarr API and quality/root profile |
| TV request never enters Sonarr | Sonarr installed and configured as default in Seerr? | Add Sonarr and test the API connection |
| Item in Radarr/Sonarr but not Deluge | Availability, monitored status, indexer/search history | Confirm suitable release and download-client test |
| Deluge at 100% but nothing in Jellyfin | Manager Activity → Queue and file location | Verify completed-download import, label and category |
| Movies import but TV episodes do not | Sonarr's category vs Deluge torrent label | Set correct label on the specific TV torrent |
| Library folder contains files but Jellyfin empty | Library type/path, scan task, user permissions | Wait for full scan; verify account visibility |
| Service resolves locally, fails remotely | Tailscale ACL/grant, Serve listener and firewall | Test an already-working tailnet service and compare ports |
| Gluetun unhealthy or torrent tracker DNS errors | Gluetun logs and DNS through VPN | Verify provider tunnel/DNS without disabling kill switch |

## Download paths and import

The media example uses:

```text
/data/Downloads/complete  # inside Deluge, Sonarr and Radarr
/data/Movies              # Radarr library
/data/TV                  # Sonarr library
/media/movies             # Jellyfin movies
/media/tv                 # Jellyfin shows
```

Check that the importer sees complete files and can write to the library:

```bash
docker exec sonarr sh -c 'test -w /data/TV && echo writable'
docker exec radarr sh -c 'test -w /data/Movies && echo writable'
docker exec sonarr sh -c 'find /data/Downloads/complete -type f | head'
docker exec jellyfin sh -c 'find /media/tv -type f | head'
```

For Deluge connectivity, the URL `http://gluetun:8112/` should resolve **inside Sonarr and Radarr**, assuming they share the same Docker network as Gluetun. An HTTP response confirms reachability, **not** that authenticated Deluge API calls work.

A Deluge *Category* configured as `tv` in Sonarr generally requires the matching **Label** in Deluge. A torrent without it may be ignored by the manager. Check **Activity → Queue** after editing the label. Separate movie and TV labels to avoid the wrong app claiming a download.

Manual import is an option for completed media that was not tracked. Prefer importing through Sonarr or Radarr and preserve seeding source files; avoid moving live torrent data.

## Jellyfin can read files but Shows is empty

1. Confirm **Dashboard → Libraries → Shows** has content type *Shows* and path `/media/tv`.
2. Run **Dashboard → Scheduled Tasks → Scan Media Library** and check that it **finishes**; large scans can take several minutes.
3. Confirm the intended Jellyfin account has access to the Shows library.
4. Check folder permissions and confirm Jellyfin can **read file contents**, not just list filenames.
5. Look at unfiltered Jellyfin logs around the scan time.

Linux paths are case-sensitive. A host path called `TV` may be mounted as `/media/tv` inside Jellyfin. That is not an error.

## Remote access fails on one port only

```bash
tailscale serve status
sudo ufw status numbered
```

Confirm the Tailscale grant/ACL permits the **specific destination and port**. An open host firewall port is not a substitute for a tailnet policy, and a Serve entry is not proof a client is authorized.

## Privacy when requesting support

Replace hostnames, internal/public IPs, device IDs, usernames and media filenames before posting diagnostics. Never publish `.env`, full `docker inspect` environment output, WireGuard profiles, API keys, auth logs with tokens, or SSH keys.

For long-term security, audit who can reach each service and restore-test backups before significant maintenance.
