# Network Services

## Pi-hole v6

- **Type:** DNS sinkhole
- **Purpose:** Blocks ads and trackers at the network level
- **Deployment:** Docker container on macvlan network
- **Address:** `10.0.1.253`
- **Web Admin:** [http://10.0.1.253/admin](http://10.0.1.253/admin)

## Omada Controller

- **Type:** Software controller
- **Purpose:** Manages TP-Link EAP 670 WiFi 6 access point
- **Deployment:** Docker container on LXC 101 (Mac Mini)
- **Web UI:** [http://10.0.1.141:8088](http://10.0.1.141:8088)

## Tailscale

- **Type:** Mesh VPN
- **Purpose:** Secure connectivity across all nodes
- **Nodes:** `alphapi5`, `alphamox`, iPhone
- **Subnet Router:** Mac Mini acts as subnet router for `10.0.1.0/24`
