# Network Infrastructure

Home network infrastructure documentation — router configuration, VLAN segmentation, DNS, and firewall rules.

## Hardware

| Component | Details |
|-----------|---------|
| Router | GL.iNet Flint 3 (BE9300) — WiFi 7 |
| Modem | Rogers XB10 (bridge mode) |
| DNS | AdGuard Home (self-hosted, `10.0.0.137`) |

---

## Topology

The Rogers XB10 operates in bridge mode, passing the public IP directly to the Flint 3. The Flint 3 handles all routing, DHCP, and DNS forwarding for the network.
```
Rogers XB10 (bridge mode — modem only)
  └── GL.iNet Flint 3 (10.0.0.1)
        ├── DHCP: 10.0.0.50 - 10.0.0.199
        └── DNS → AdGuard Home (10.0.0.137)
                      └── Upstream: 1.1.1.1 / 1.0.0.1 (Cloudflare)
```

---

## IP Assignments

| Host | IP | Role |
|------|----|------|
| Flint 3 | 10.0.0.1 | Router / DHCP / DNS forwarder |
| Proxmox | 10.0.0.234 | Hypervisor |
| Mediastack LXC | 10.0.0.136 | Reverse proxy, Docker host |
| AdGuard LXC | 10.0.0.137 | DNS, ad blocking, Tailscale |
| Jellyfin LXC | 10.0.0.144 | Media server |
| Frost (Windows) | 10.0.0.192 | Media storage / SMB share |

All critical devices have static IP reservations configured on the Flint 3.

---

## DNS

AdGuard Home handles all DNS for the network, enforced at the router level.

- **Upstream DNS:** Cloudflare 1.1.1.1 and 1.0.0.1 (plain DNS, parallel requests)
- **Fallback DNS:** 1.1.1.1, 1.0.0.1
- **Upstream timeout:** 3 seconds
- **Wildcard rewrite:** *.idiotproductions.net → 10.0.0.136
- **Bootstrap DNS:** 1.1.1.1, 1.0.0.1, 8.8.8.8

### AdGuard LXC DNS config
Tailscale is prevented from overwriting /etc/resolv.conf on the AdGuard LXC:
tailscale set --accept-dns=false

/etc/resolv.conf on the AdGuard LXC:
nameserver 1.1.1.1
nameserver 1.0.0.1

---

## VLANs

In progress — being configured on Flint 3

Planned VLAN segmentation:

| VLAN | ID | Subnet | Purpose |
|------|----|--------|---------|
| Trusted | 10 | 10.10.0.0/24 | Main devices, homelab |
| IoT | 20 | 10.20.0.0/24 | Smart TVs, consoles, untrusted devices |
| Guest | 30 | 10.30.0.0/24 | Guest WiFi, internet only |

---

## Flint 3 DNS Settings

| Setting | Value |
|---------|-------|
| Mode | Manual DNS |
| DNS Server 1 | 10.0.0.137 (AdGuard) |
| Override DNS of All Clients | OFF |
| DNS Rebinding Protection | Off |

Do not enable Override DNS Settings of All Clients — this intercepts all outbound port 53 traffic including AdGuard's own upstream queries, creating a DNS loop that breaks resolution network-wide.

---

## Related

- homelab-infrastructure: https://github.com/TechAsura/homelab-infrastructure
