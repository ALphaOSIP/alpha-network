# Alpha Network

## Overview

The **alpha-network** is a homelab network operating on the `10.0.1.0/24` subnet. It serves as the backbone for self-hosted services, media streaming, home automation, and lab experimentation.

**Subnet:** `10.0.1.0/24`  
**Default Gateway:** `10.0.1.1` (OPNsense)  
**DHCP:** OPNsense  
**DNS:** Pi-hole (`10.0.1.253`)  
**Public IP:** Dynamic (assigned by ISP via fiber ONT)

---

## Physical Topology

```
              Internet
                 │
            [ONT - Fiber Modem]
                 │
         [OPNsense - Dell OptiPlex]
          │                   │
    10.0.1.0/24 LAN         WAN (ISP)
          │
    [TL-SG108E Switch]
     /     │   │    │     \
    │      │   │    │      │
Latitude Pi 5 Mac Mini EAP 670  Pi 3
 .176   .100  .108   .157  .158
 🔴OFF
   (offline since 2026-08-20)
   ├── ALphaMAIN .233 (NEW Aug 2026 — was .141; usually off)
   ├── ALpha-Server .135 (NEW Aug 2026 — Frigate NVR)
   ├── Pi-hole .253 (macvlan DNS)
   └── Docker services (light — *arr, Immich, zigbee, MQTT)
```

> **⚠️ 2026-09-13:** The Latitude (`.176`) is still **offline since ~2026-08-20 03:25 EDT** — Jellyfin/RomM/n8n/FreshRSS/Homepage/Dozzle are DOWN. **Immich has been migrated back to the Pi 5 (`.100:2283`) and is healthy again since 2026-09-08.** ALpha-Server (`.135`) still runs only Frigate.

---

## Hardware & Roles

| Device | Model / Spec | Role |
|--------|-------------|------|
| **OPNsense** | Dell OptiPlex | Router, DHCP server, stateful firewall |
| **ALphaMAIN** | ASUS (model TBD) | Personal gaming PC — SSH-monitor only, no agents (added Aug 2026, IP .233) |
| **ALpha-Server** | Custom AMD (Ryzen 5 3600, RX 580) | Frigate NVR / GPU box — Ubuntu 24.04 (added Aug 2026) |
| **alphamobile-1** | Dell Latitude 5501 (i7-9850H) | Heavy lifter — transcoding, ML, DBs — 🔴 OFFLINE since 2026-08-20 (Immich migrated back to Pi 5) |
| **alphapi5** | Raspberry Pi 5 | Orchestration — *arr, Zigbee, DNS, **Immich (photos)** |
| **alphapi3** | Raspberry Pi 3 | Secondary node (travel stick plans) |
| **alphamox** | Mac Mini | Proxmox hypervisor, HA VM |
| **TL-SG108E** | TP-Link 8-port | Managed Gigabit switch |
| **EAP 670** | TP-Link Omada | WiFi 6 access point (PoE powered, standalone) |

---

## IP Assignments

| Device | IP | Purpose |
|--------|-----|---------|
| OPNsense | `10.0.1.1` | Router, DHCP, firewall |
| ALphaMAIN | `10.0.1.233` | **NEW Aug 2026** — main machine (ASUS); DHCP moved from `.141` (~Aug 20) |
| ALpha-Server | `10.0.1.135` | **NEW Aug 2026** — Frigate NVR (Ryzen 5 3600, Ubuntu 24.04) |
| alphamobile-1 | `10.0.1.176` | Latitude heavy lifter — 🔴 OFFLINE since ~Aug 20 2026 (services down; Immich migrated to Pi 5) |
| alphapi5 | `10.0.1.100` | Orchestration — *arr, Zigbee, DNS, RDTClient, **Immich** |
| alphapi3 | `10.0.1.158` | Secondary node (travel stick plans) |
| HP Printer 1 | `10.0.1.103` | Office printer |
| alphamox | `10.0.1.108` | Proxmox hypervisor |
| HP Printer 2 | `10.0.1.116` | Secondary printer |
| LG TV | `10.0.1.138` | Smart TV (LG Innotek OUI — was .143) |
| HA VM | `10.0.1.154` | Home Assistant |
| Eufy HomeBase | `10.0.1.88` | Camera bridge (DHCP moved from `.187` — verified 2026-09-20) |
| Pi-hole | `10.0.1.253` | DNS resolver (macvlan container on Pi 5) |
| EAP 670 | `10.0.1.216` | WiFi access point (DHCP moved from `.157` — verified 2026-09-20) |

> **Note (2026-09-20):** Two documented devices picked up new DHCP leases this week — the EAP 670 (`.157` → `10.0.1.216`) and the Eufy HomeBase (`.187` → `10.0.1.88`). Neither is statically reserved on OPNsense, so re-check the ARP/lease table if a service can't reach them.

---

## VLANs (configured Aug 2026)

OPNsense has four VLAN interfaces (confirmed live 2026-08-09). No static mappings yet — devices must be moved/assigned explicitly.

| VLAN | Name | Subnet | OPNsense IP |
|------|------|--------|-------------|
| vlan01 | Trusted | `10.0.10.0/24` | `10.0.10.1` |
| vlan02 | Services | `10.0.20.0/24` | `10.0.20.1` |
| vlan03 | IoT | `10.0.30.0/24` | `10.0.30.1` |
| vlan04 | Guest | `10.0.40.0/24` | `10.0.40.1` |

> All four gateways serve the OPNsense web UI on :80/:443. As of 2026-08-09 no LAN device has been moved to a VLAN yet (everything still on the flat 10.0.1.0/24).

---

## DHCP

- **Server:** OPNsense (ISC DHCP or Kea, depending on OPNsense version)
- **Scope:** `10.0.1.50` – `10.0.1.254`
- **Reservations:** All static IPs above are configured as DHCP reservations by MAC address
- **⚠️ 2026-08-23 note:** reservations are NOT fully enforced — ALphaMAIN's lease moved `.141 → .233` despite the docs listing `.141` (OPNsense unreachable from the cron host to verify, so treat DHCP IPs as best-effort).
- **Lease time:** 24 hours (default)
- **DNS servers advertised:** `10.0.1.253` (Pi-hole) — single server handed out to force all DNS through Pi-hole

### Adding a new reservation
1. Log into OPNsense → **Services > DHCP Server > LAN**
2. Scroll to **DHCP Static Mappings**
3. Click **+** — enter MAC address, IP address, hostname
4. Optionally set a static ARP entry for extra protection against IP spoofing

---

## DNS

- **Server:** Pi-hole v6 (containerized on alphapi5)
- **Address:** `10.0.1.253`
- **Network mode:** macvlan — has its own IP on the LAN, bypasses host networking
- **Upstream DNS:** Cloudflare (`1.1.1.1`, `1.0.0.1`) or Quad9 (`9.9.9.9`) — set via Pi-hole admin UI
- **Local domain:** `alpha.lan` (optional; conditional forwarding not currently configured)
- **Admin UI:** `http://10.0.1.253/admin` (password in vault)

All LAN clients receive Pi-hole as their only DNS server via DHCP. This ensures ad/tracker blocking applies network-wide. For devices that bypass DHCP (e.g., static IPs), configure DNS manually to `10.0.1.253`.

### Adding local DNS records
Pi-hole can be used for local name resolution via **Local DNS Records** in the admin panel. Alternatively, use OPNsense's **Unbound DNS** with **Host Overrides** if you prefer the router handle it. Currently, the network relies on IP addresses rather than hostnames for internal access.

---

## Firewall

### Edge (OPNsense)
- **Stateful inspection** enabled on all interfaces
- **Default policy:** Block inbound from WAN, allow all outbound
- **Anti-lockout rule:** Enabled (port 22/443 on LAN always accessible)
- **NAT:** Outbound NAT (automatic) for LAN → WAN traffic
- **Port forwards:** None on standard ports. Any external access goes through Tailscale.

### Host-level (UFW)

**alphapi5 (Pi 5)**
```
Status: active
To                         Action      From
--                         ------      ----
22/tcp (SSH)               ALLOW       10.0.1.0/24
9000/tcp (Portainer)       ALLOW       10.0.1.0/24
3000/tcp (RomM)            ALLOW       10.0.1.0/24
8096/tcp (Jellyfin)        ALLOW       10.0.1.0/24
3001/tcp (Uptime Kuma)     ALLOW       10.0.1.0/24
2283/tcp (Immich)          ALLOW       10.0.1.0/24
Anywhere                   ALLOW       Tailscale (100.x.y.z)
```

**alphamox (Mac Mini / Proxmox)**
```
Status: active
To                         Action      From
--                         ------      ----
22/tcp (SSH)               ALLOW       10.0.1.0/24
8006/tcp (Proxmox Web UI)  ALLOW       10.0.1.0/24
Anywhere                   ALLOW       Tailscale (100.x.y.z)
```

---

## Remote Access

### Tailscale (Primary)

Tailscale is the primary remote access method — a WireGuard-based mesh VPN with automatic NAT traversal.

| Node | Hostname | Tailscale IP |
|------|----------|-------------|
| alphapi5 | alphapi5 | `100.101.94.73` |
| alphamox | alphamox | `100.124.155.110` |
| ALpha-Server | alpha-server | `100.123.100.38` |
| alphamobile-1 | alphamobile-1 | `100.82.167.20` — 🔴 offline, last seen 2026-08-20 |
| iPhone | — | `100.89.238.113` |

**Subnet routing:** The Mac Mini (alphamox) is configured as a **subnet router** for `10.0.1.0/24`. This allows remote devices (e.g., iPhone) to reach LAN-only services like Home Assistant (`10.0.1.154:8123`) without exposing them to the internet.

**Setup summary:**
1. Install Tailscale on each node via `curl -fsSL https://tailscale.com/install.sh | sh`
2. Authenticate each node: `tailscale up`
3. On the subnet router: `tailscale up --advertise-routes=10.0.1.0/24 --accept-routes`
4. Approve the advertised routes in the [Tailscale Admin Console](https://login.tailscale.com) under **Machines > Edit Route Settings**

> **⚠️ Important routing fix:** On nodes that are *not* the subnet router (e.g., alphapi5), set `--accept-routes=false` to prevent Tailscale from routing local subnet traffic through the mesh tunnel instead of the LAN. Without this, traffic to `10.0.1.x` destinations gets sent through `tailscale0` instead of `eth0`, breaking local communication. This was a hard-learned lesson — the Pi 5 became unreachable from the LAN because all return traffic was going out through the Tailscale tunnel.

### WireGuard (Fallback)

WireGuard is configured on OPNsense but is **not actively used**. Tailscale replaced it for all remote access — no need to maintain port forwards or dynamic DNS when using Tailscale.

---

## WiFi

- **Access Point:** TP-Link EAP 670 (WiFi 6, 2.4/5 GHz) at `10.0.1.216` (was `.157` until ~Sep 2026)
- **Power:** PoE (via included PoE injector or PoE switch — currently via injector)
- **Management:** Standalone mode (Omada controller removed Aug 2026 — EAP runs autonomously with cached config)

### SSIDs

| SSID | Band | Purpose |
|------|------|---------|
| `Prince Network` | 2.4/5 GHz | Main network, bridged to LAN, clients get DHCP from OPNsense |

> No VLANs are configured for WiFi clients yet — the EAP bridges to the flat `10.0.1.0/24` network. (OPNsense-side VLANs exist as of Aug 2026, see [VLANs](#vlans-configured-aug-2026) — nothing is routed to them yet.)

---

## Switch

- **Model:** TP-Link TL-SG108E (8-port Gigabit)
- **Mode:** Unmanaged (operating as a simple dumb switch)
- **VLANs:** Not configured
- **Connections:**
  - Port 1: OPNsense LAN (uplink)
  - Port 2: alphapi5
  - Port 3: alphamox (Proxmox)
  - Port 4: EAP 670 (PoE injector)
  - Port 5: alphapi3 (offline)
  - Port 6–8: Open / future expansion

The "Easy Smart" features of this switch are unused; it functions purely as an unmanaged gigabit switch.

---

## Key Services (alphapi5)

The Raspberry Pi 5 (`10.0.1.100`) runs Docker with the following LAN services:

| Service | URL | Port |
|---------|-----|------|
| Portainer | `http://10.0.1.100:9000` | Docker management |
| Uptime Kuma | `http://10.0.1.100:3001` | Uptime monitoring |
| Sonarr / Radarr / Prowlarr / Bazarr | `http://10.0.1.100:8989/7878/9696/6767` | *arr media automation |
| Zigbee2MQTT + Mosquitto | `10.0.1.100:8080/1883` | Zigbee bridge + MQTT |
| RDTClient | `http://10.0.1.100:6500` | Real-Debrid download client |
| Immich | `http://10.0.1.100:2283` | Photo library (returned from the Latitude 2026-09-08; + Postgres, Redis, ML containers) |
| go2rtc | `rtsp://10.0.1.100:8554` | RTSP restreamer (Eufy/camera streams → HA + Frigate) |

> **⚠️ 2026-09-13:** **Immich now runs on the Pi 5 again** (`10.0.1.100:2283`, 4 healthy containers, migrated from the offline Latitude 2026-09-08 — library on the NAS at `/mnt/nas-data/immich`). Jellyfin / RomM / n8n / FreshRSS are **still DOWN** with the **Latitude (`.176`)**, which has been **offline since ~2026-08-20**. See [HARDWARE.md](HARDWARE.md) for the full outage trail.

---

## Maintenance Notes

### Rebooting OPNsense
- SSH to `10.0.1.1` or use the web UI at `https://10.0.1.1`
- Command: `sudo reboot`
- Expect ~2 min downtime during boot — all LAN traffic halts until the firewall is back up

> **⚠️ 2026-09-06:** SSH (port 22) to `10.0.1.1` has been **unreachable from alphapi5 and ALpha-Server** for several weeks (connection timeout — the anti-lockout rule allows it, so the OPNsense SSH daemon itself may be stopped). Use the **web UI** (`https://10.0.1.1`) instead. This is why the weekly check's DHCP-lease fetch has been failing — see [CHANGELOG.md](CHANGELOG.md).

### Pi-hole updates
- Run inside the Docker container: `docker exec pihole pihole -up`
- Gravity (blocklist) update: `docker exec pihole pihole updateGravity`
- Web UI: `http://10.0.1.253/admin`

### Adding a new device to the network
1. Physically connect (Ethernet) or connect to the `Prince Network` SSID (WiFi)
2. Device gets a DHCP lease in the `10.0.1.50–254` range
3. Optionally create a **DHCP reservation** in OPNsense for a static IP
4. Point DNS to `10.0.1.253` manually if the device uses static networking
5. **Document it** — add to [HARDWARE.md](HARDWARE.md) / [NETWORK.md](NETWORK.md) so the weekly cron and monitoring know it exists

### Troubleshooting connectivity
1. **Is the device on the right network?** Check that WiFi SSID is `Prince Network` (not guest)
2. **Does it have an IP?** Run `ip a` (Linux) or `ipconfig` (Windows) — should be `10.0.1.x`
3. **Can it reach the gateway?** `ping 10.0.1.1`
4. **DNS working?** `nslookup google.com 10.0.1.253` — should resolve
5. **Check Pi-hole query log** at `http://10.0.1.253/admin/query_log.php` — look for blocked queries
6. **Check OPNsense firewall logs** at **Firewall > Log Files**

---

## Future Improvements

- [ ] Configure **VLANs** for IoT device isolation
- [ ] Enable **mDNS reflector** on OPNsense for service discovery across subnets
- [ ] Set up **conditional forwarding** in Pi-hole for `alpha.lan` domain resolution
- [ ] Replace flat switch with a **managed PoE switch** (e.g., TL-SG2008P) for VLAN trunking
- [ ] Implement **automatic backup** of OPNsense config to alphapi5
- [ ] Add a **fallback internet connection** (4G LTE failover via USB modem on OPNsense)
- [ ] Decide Latitude fate: power it back on, or migrate the remaining services (Jellyfin/n8n/FreshRSS/RomM) to the Pi 5 / ALpha-Server and retire it — **Immich already migrated back to the Pi 5 (2026-09-08)**
