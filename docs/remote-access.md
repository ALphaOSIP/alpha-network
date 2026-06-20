# Remote Access

## Overview

Remote access to the homelab is achieved primarily through **Tailscale**, a WireGuard-based mesh VPN that provides secure, zero-configuration connectivity across all devices. A WireGuard instance on OPNsense exists as a fallback but has been superseded by Tailscale for daily use.

---

## Tailscale Mesh VPN

Tailscale is installed on the following devices:

| Device       | Role                        |
|--------------|-----------------------------|
| alphapi5     | Homelab node / client       |
| alphamox     | macOS workstation           |
| iPhone       | On-the-go mobile access     |

Each device joins the same Tailscale network (tailnet) and can reach any other peer directly via a secure WireGuard tunnel, without any open inbound ports.

---

## Subnet Routing

The **Mac Mini** acts as a **Tailscale subnet router**, advertising the local LAN subnet:

```
Advertised route: 10.0.1.0/24
```

This allows Tailscale peers outside the local network (e.g., the iPhone on cellular) to reach devices on the `10.0.1.0/24` LAN as if they were locally connected, without installing Tailscale on every LAN device.

### Enabling subnet routing on the Mac Mini

```bash
sudo tailscale up --advertise-routes=10.0.1.0/24
```

Routes must then be approved in the Tailscale admin console.

---

## Accessing Home Assistant Remotely

Home Assistant runs on the LAN at `10.0.1.154:8123`. To access it remotely:

1. Connect to Tailscale on your remote device (e.g., iPhone, laptop).
2. Traffic destined for `10.0.1.0/24` is routed through the Mac Mini subnet router.
3. Navigate to `http://10.0.1.154:8123` in your browser.

No port forwarding or public DNS is required — all traffic stays within the encrypted Tailscale mesh.

---

## Routing Fix: `--accept-routes=false` on alphapi5

To prevent local network disruption on `alphapi5`, Tailscale is configured **not** to accept routes from other peers:

```bash
sudo tailscale up --accept-routes=false
```

This ensures that `alphapi5` does not import the `10.0.1.0/24` route from the Mac Mini's subnet advertisement, avoiding potential routing conflicts or overlaps with the local subnet. The device still participates in the tailnet for direct peer-to-peer connectivity.

---

## Tailscale Funnel / DERP Relays

- **Funnel** (public HTTPS ingress): Not configured.
- **DERP relays** (relay fallback when direct connections fail): Not explicitly configured; Tailscale's default DERP servers are used if needed, but direct peer-to-peer connections are preferred.

All remote access relies on Tailscale's mesh routing rather than public-facing endpoints.

---

## WireGuard (OPNsense) — Fallback

A **WireGuard** tunnel is configured on the **OPNsense** firewall as a backup remote-access mechanism. This predates Tailscale adoption and is maintained for redundancy.

- Protocol: WireGuard
- Host: OPNsense (acts as WireGuard server)
- Client configs: Limited set of fallback devices
- Status: **Not in daily use** — Tailscale handles all routine remote access

---

## Summary

| Method        | Primary | Encryption | Ports Exposed | Use Case                     |
|---------------|---------|------------|---------------|------------------------------|
| Tailscale     | Yes     | WireGuard  | None          | Daily remote access          |
| WireGuard     | No      | WireGuard  | 51820/UDP     | Fallback / redundancy        |

All remote traffic is encrypted end-to-end. No services are exposed directly to the public internet.
