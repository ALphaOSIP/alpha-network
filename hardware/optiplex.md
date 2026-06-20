# Dell OptiPlex (Small Form Factor)

## Overview

The Dell OptiPlex (small form factor) serves as the primary router and firewall for the home network, running **OPNsense**. It handles all routing, NAT, DHCP, and firewall duties for the 10.0.1.0/24 subnet.

## Network Role

| Attribute | Value |
|-----------|-------|
| **Role** | Router / Firewall |
| **OS** | OPNsense |
| **LAN IP** | 10.0.1.1 (Gateway) |
| **WAN** | ISP ONT connection |
| **LAN** | Feeds TP-Link TL-SG108E switch |
| **DHCP** | 10.0.1.0/24, lease time 24h |
| **Uptime** | 57+ days |

## Services

- **DHCP Server** — Assigns addresses in 10.0.1.0/24 with a 24-hour lease time
- **WireGuard** — Configured but not actively used; Tailscale is the preferred mesh VPN solution
- **Firewall / NAT** — OPNsense default rules with stateful inspection

## Connectivity

```
ISP ONT → OptiPlex (WAN) → OptiPlex (LAN) → TL-SG108E → LAN clients
```

## Notes

- Tailscale is used on the LAN side for remote access and inter-device mesh connectivity; WireGuard remains as a fallback option.
- The device has been online continuously for 57+ days at last check, indicating stable operation.
- All LAN traffic is NAT'd through the WAN interface to the ISP optical network terminal.
