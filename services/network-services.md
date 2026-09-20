# Network Services

## Pi-hole v6

- **Type:** DNS sinkhole
- **Purpose:** Blocks ads and trackers at the network level
- **Deployment:** Docker container on macvlan network
- **Address:** `10.0.1.253`
- **Web Admin:** [http://10.0.1.253/admin](http://10.0.1.253/admin)

## EAP 670 (Standalone)

- **Type:** WiFi 6 access point (TP-Link EAP 670)
- **Purpose:** Main WiFi — SSID `Prince Network` (2.4/5 GHz)
- **Management:** Standalone mode — Omada controller removed Aug 2026, EAP runs with cached config
- **Address:** `10.0.1.216` (was `.157` until ~Sep 2026)

## Tailscale

- **Type:** Mesh VPN
- **Purpose:** Secure connectivity across all nodes
- **Nodes:** `alphapi5`, `alphamox`, iPhone
- **Subnet Router:** Mac Mini acts as subnet router for `10.0.1.0/24`
