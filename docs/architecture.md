# Architecture and design decisions

A maintainable homelab separates **access**, **applications**, and **data**. The containers can change independently; persistent folders and network rules need deliberate management.

## Browser access

```text
Client -> LAN or Tailscale -> private DNS -> Caddy HTTPS -> application
```

- Private DNS maps a service name (for example `home.lab.test`) to a **reachable** proxy address.
- Caddy terminates HTTPS and forwards to the appropriate app. Caddy and the app must share a Docker network or use a reachable host port.
- Prefer internal networks and loopback-only published ports for admin interfaces.
- Tailscale provides private reachability; its ACLs/grants and host firewall still decide who can use each service.
- If you use Tailscale Serve instead of Caddy, do not make both own the same listener.

The [basic Compose example](../examples/docker/compose.example.yml) puts Caddy and its upstreams on the same Docker network. The [media example](../examples/media/compose.example.yml) binds individual app web UIs to loopback. They are **independent demonstrations**, not one ready-made combined deployment.

## Media data flow

```text
Seerr request
   |--> Radarr -> movie management --+
   \--> Sonarr -> series management -+-> Deluge (via Gluetun)
                    ^                     |
                    | Prowlarr indexers    | completed files
                    +----------------------+ 
                                           |
                        Radarr/Sonarr completed-download import
                                  |               |
                              /data/Movies    /data/TV
                                  \              /
                                         Jellyfin
```

The arrows from Prowlarr describe indexer synchronization, **not** a download transfer through Prowlarr. Sonarr and Radarr send jobs to the download client, then track and import completed files.

## Shared paths and Docker mounts

Keep download and library folders underneath a shared host root, mounted **at the same container path** for Deluge, Sonarr and Radarr:

```text
Host:       /absolute/path/to/media/Downloads
            /absolute/path/to/media/Movies
            /absolute/path/to/media/TV

Download/import containers: /data/Downloads
                            /data/Movies
                            /data/TV

Jellyfin:                  /media/movies
                           /media/tv
```

Jellyfin's paths may differ because they are separate bind mounts. Path names are case-sensitive on Linux: `TV` on the host can intentionally appear as `/media/tv` inside Jellyfin.

Sharing a filesystem can support hardlinks instead of making extra copies when importing seeding torrents. Configure permissions consistently without making the directories world-writable.

## VPN isolation

Deluge can use `network_mode: service:gluetun` so its network traffic shares Gluetun's VPN namespace. Publish the Deluge Web UI port **on Gluetun**, not on Deluge. Sonarr and Radarr remain on the normal internal Docker network and connect to the Deluge Web UI using `gluetun:8112`.

Do not route all management applications through the download VPN or disable Gluetun's firewall to solve an unrelated connectivity problem.

For setup steps and common failure modes, see [media automation](media-automation.md) and [troubleshooting](troubleshooting.md).
