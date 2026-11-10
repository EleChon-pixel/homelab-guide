# Private remote access with Tailscale

Tailscale allows enrolled devices to reach a server without opening general internet-facing application ports. Use it with **app authentication**, narrow **grants/ACLs**, and host firewall rules; joining a tailnet is not permission to access everything.

## Choose an access pattern

| Pattern | Use case | Notes |
| --- | --- | --- |
| Private DNS + Caddy HTTPS | Consistent service names on LAN and tailnet | Client must resolve the name and trust the certificate |
| Tailscale Serve HTTPS | An individual private service reached through a tailnet HTTPS listener | Configure an unused port, check Serve status and tailnet grants |
| SSH local forwarding | Temporary access via an authorized computer | Restrict forwarded destination and protect the client port |
| Tailnet-aware router / subnet gateway | TVs and devices without a native Tailscale app | Requires explicit routing/firewall setup and careful access controls |

**Tailscale Serve and Caddy must not conflict over the same listener.** A running service is not proof that the remote client can access its port.

## Typical checks

```bash
tailscale status
tailscale serve status
sudo ufw status numbered
```

If one Serve port loads on a phone but another does not, inspect **Tailscale grants/ACLs for the exact destination port** before changing Docker or the firewall. Then confirm the Serve upstream responds locally on the host. A host UFW allow rule does not override a Tailscale access denial.

Tailscale Serve can publish an HTTPS endpoint privately within a tailnet. This is not the same as intentionally publishing a service to the whole internet (Tailscale Funnel or traditional router forwarding).

## A TV without a native VPN client

Some TVs cannot run Tailscale natively. Rather than root the device, a trusted laptop on the TV's local Wi-Fi can forward a Jellyfin service over a Tailscale-protected SSH connection. The TV connects to the laptop's local port.

Keep in mind:
- The laptop must remain powered on and the SSH tunnel active while streaming.
- Restrict the listener to the expected local interface and use a firewall rule that allows only the TV.
- If the TV-to-laptop segment is plain HTTP, its contents and login credentials are **not encrypted on that segment**. Use only on a trusted LAN; prefer HTTPS for stronger protection.
- A dedicated SSH key should be restricted to forwarding, not general shell access, when possible.
- Do not expose a loopback-bound service to a wider network without reviewing authentication and firewall policy.

Never put real tailnet DNS names, device identities, addresses, SSH keys or ACLs for your production environment in a public guide.

For local HTTPS, see [Caddy](reverse-proxy.md). For deeper host security, see [security](security.md).
