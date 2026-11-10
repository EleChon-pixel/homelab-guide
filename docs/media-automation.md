# Media automation: Jellyfin + Seerr + Radarr + Sonarr

This optional stack lets a household request content, manage a library, and watch completed media. Use it only for media you own or are authorized to download. You do **not** need torrents or a VPN to run Jellyfin.

## 1. Understand the roles

```text
Seerr
  ├─ movie request ──> Radarr ──┐
  └─ series request ─> Sonarr ──┤
                        ^      │
       Prowlarr indexer sync   │
                               v
                         Deluge Web API
                               │
                 Deluge traffic via Gluetun VPN
                               │
                /data/Downloads/complete
                               │
                Radarr / Sonarr import
                  ├─ /data/Movies
                  └─ /data/TV
                               │
                            Jellyfin
```

Seerr is the request interface, **not** the download client. Prowlarr manages indexers but does not transfer media. Radarr and Sonarr add downloads, monitor their completion and import them into the library.

## 2. Prepare storage

Choose your own persistent host directory. For the [example Compose file](../examples/media/compose.example.yml), create a separate working directory and use this layout:

```text
media-demo/
  compose.yml
  .env                  # local, ignored; never commit
  secrets/wg0.conf      # real VPN profile; never commit
  state/                # application data; never commit
  media/
    Downloads/complete/
    Downloads/incomplete/
    Movies/
    TV/
```

Create required folders with appropriate user ownership:

```bash
mkdir -p media/{Downloads/complete,Downloads/incomplete,Movies,TV}
mkdir -p state/{gluetun,deluge,radarr,sonarr,prowlarr,seerr,jellyfin/config,jellyfin/cache}
mkdir -p secrets
chmod 700 secrets
```

All of **Deluge, Radarr and Sonarr** mount the same host `media` directory at `/data`. A completed torrent at `/data/Downloads/complete` is therefore visible from the importers at exactly that path. Jellyfin reads `/media/movies` and `/media/tv` using separate readonly mounts, which is intentional.

The example uses `PUID` and `PGID` for LinuxServer containers. Set these to an account with access to the folders. Do not apply blanket `chmod 777` fixes.

## 3. Configure VPN isolation before launching Deluge

The [media example](../examples/media/compose.example.yml) assumes **a custom WireGuard configuration file** for Gluetun. Obtain a valid profile from a provider you trust and store it *only* at `secrets/wg0.conf` in your private working directory. This repository **does not contain a real profile** or any key material.

Gluetun supports loading a WireGuard ini file mounted at `/gluetun/wireguard/wg0.conf`; the profile's values take precedence over conflicting environment settings. Confirm the provider's configuration and Gluetun version in the [official custom-provider documentation](https://github.com/qdm12/gluetun-wiki/blob/main/setup/providers/custom.md).

The example sets `VPN_SERVICE_PROVIDER=custom`, `VPN_TYPE=wireguard`, and places Deluge in `network_mode: service:gluetun`. Deluge's exposed port belongs to **Gluetun**, not Deluge. Avoid disabling the VPN firewall/kill switch to debug connectivity.

For VPN providers that support **port forwarding**, read the [Gluetun guide](https://github.com/qdm12/gluetun-wiki/blob/main/setup/advanced/vpn-port-forwarding.md). This is a VPN-side port, **not** a router port-forwarding rule. For Proton WireGuard, the generated provider configuration must have NAT-PMP enabled and the target server must support the feature. A provider-assigned port may change after reconnection; synchronize Deluge's incoming listening port accordingly.

## 4. Start and secure the apps

Copy both example files into a separate directory (do not run the examples in Git):

```bash
mkdir -p ../media-demo
cp examples/media/compose.example.yml ../media-demo/compose.yml
cp examples/media/.env.example ../media-demo/.env
cd ../media-demo

# Create the folders shown above, choose your own PUID/PGID and timezone,
# and privately place a real WireGuard profile at secrets/wg0.conf.
docker compose config --quiet
docker compose up -d
docker compose ps
```

All sample web ports bind to **127.0.0.1**. Use a separately secured local reverse proxy or a private Tailscale access path to open the UIs from another device; don't publish administrative services directly to the internet.

Create strong application logins. Back up configs and avoid sharing API keys.

## 5. Connect Prowlarr, download clients and Seerr

Configure the stack **in this order**:

1. In **Prowlarr → Settings → Apps**, add Radarr and Sonarr. Inside the example Docker network use `http://radarr:7878` and `http://sonarr:8989`. Enter each app's API key locally. Keep **API authentication** separate from login form credentials.
2. In **Radarr → Settings → Download Clients**, add Deluge using host `gluetun` and port `8112`; test the connection. Repeat in **Sonarr**. They must talk to Gluetun because Deluge shares Gluetun's network namespace.
3. Give Radarr and Sonarr **distinct categories/labels** (for example `movies` and `tv`); enable and configure Deluge's **Label** plugin to support them. Ensure torrents in Deluge use the corresponding label so the right app tracks imports.
4. Set **Radarr root folder** to `/data/Movies` and **Sonarr root folder** to `/data/TV`. Enable Completed Download Handling in both apps and verify the folders are writable.
5. In **Seerr → Settings → Services**, add Radarr at `radarr:7878` and Sonarr at `sonarr:8989`. Select **default servers**, valid quality profiles and the root folders above; enable automatic search if desired and configure request approval permissions.
6. In **Jellyfin**, create *Movies* pointing to `/media/movies` and *Shows* pointing to `/media/tv`. Give intended users library access and schedule or enable automatic library scans.

Requests made before an app was configured may need review or a new manual search. Don't assume they're replayed automatically.

## 6. Verify the entire chain

Use a small, authorized test item.

| Stage | What success looks like |
| --- | --- |
| Seerr | Request approved and sent to the default manager |
| Radarr / Sonarr | Item appears as monitored; search may be in progress |
| Deluge | Item downloads; seeding can continue after 100% |
| Radarr / Sonarr | Video file appears inside `/data/Movies` or `/data/TV` |
| Jellyfin | Library scan recognizes the imported file and the user can view it |

**100% in Deluge does not mean imported.** A finished torrent can keep seeding while the manager uses a hardlink or copy to add it to the library.

If the download is present but not imported, check **Activity → Queue** in the relevant manager for import warnings and confirm the Deluge **label** matches the app's configured category. Check `/data/Downloads/complete` for the actual files. Avoid moving or deleting seeding torrents manually.

If imported episodes are present but Jellyfin appears empty, verify the **Shows** library type and path, the account's **library access**, and that **Scan Media Library** completed. A large initial scan may take minutes. See [troubleshooting](troubleshooting.md).

## 7. Hardlinks, upgrades, and housekeeping

When completed downloads and libraries share a filesystem and compatible mount paths, the importer can hardlink a seeding file into the library without duplicating its data. Moving or renaming a file out of Deluge's managed download folder can break seeding. Keep downloaded-file cleanup separate from the import process.

Keep Docker images updated thoughtfully, back up application configuration before major updates, and review VPN port-forwarding state after reconnects. Never commit `.env`, `secrets/`, actual WireGuard profiles, application databases or API keys.

**Useful references:** [Sonarr](https://wiki.servarr.com/sonarr), [Radarr](https://wiki.servarr.com/radarr), [Jellyfin](https://jellyfin.org/docs/), [Gluetun](https://github.com/qdm12/gluetun-wiki).
