# 🖥️ Alpha-Network Hardware Inventory

> Last updated: 2026-09-06

This document catalogs all hardware in the Alpha homelab network (10.0.1.0/24).

---

## 📋 Summary

| Device | Role | OS | IP | CPU | RAM | Status |
|--------|------|----|----|-----|-----|--------|
| **ALphaMAIN** | Personal gaming PC | Windows | 10.0.1.233 | TBD | TBD | 🟡 Personal machine, SSH-monitor only — IP moved from .141 (Aug 2026) |
| **ALpha-Server** | Frigate NVR / GPU box | Ubuntu 24.04 | 10.0.1.135 | Ryzen 5 3600 (6C/12T) | 16 GB | ✅ Online — new (Aug 2026) |
| **alphamobile-1 (Latitude)** | Heavy lifter server | Ubuntu 24.04 LTS | 10.0.1.176 | i7-9850H (6C/12T) @ 4.6 GHz | ~7.5 GB | 🔴 OFFLINE since 2026-08-20 — services down |
| **alphapi5** | Lightweight orchestration server | Ubuntu 24.04.1 LTS | 10.0.1.100 | 4× Cortex-A76 @ 2.4 GHz | 8 GB | ✅ Online |
| **alphamox** | Proxmox hypervisor | Proxmox VE 9.1.1 | 10.0.1.108 | i5-4260U (2C/4T) @ 1.4 GHz | 8 GB DDR3 | ✅ Online |
| **OPNsense (Dell OptiPlex)** | Router / Firewall | OPNsense | 10.0.1.1 | — | — | ✅ Online |
| **alphapi3** | Secondary node (travel stick plans) | Ubuntu 24.04 | 10.0.1.158 | 4× Cortex-A53 @ 1.4 GHz | 1 GB | ✅ Online (was .101, now .158) |
| **TL-SG108E** | Managed switch | — | — | — | — | ✅ Online |
| **EAP 670** | WiFi 6 AP | — | — | — | — | ✅ Online |

---

## 🧩 alphapi5 — Raspberry Pi 5

| Spec | Detail |
|------|--------|
| **Model** | Raspberry Pi 5 |
| **RAM** | 8 GB LPDDR4X |
| **CPU** | 4× ARM Cortex-A76 @ 2.4 GHz |
| **Kernel** | 6.8.0-1052-raspi |
| **OS** | Ubuntu 24.04.1 LTS |
| **MAC Address** | `2c:cf:67:27:49:1a` |
| **Network** | Gigabit Ethernet (eth0) |
| **Uptime** | 57+ days (rock solid) |
| **Temperature** | ~61 °C idle |

### Storage

| Device | Type | Capacity | Used | Available |
|--------|------|----------|------|-----------|
| SD Card | microSD | 256 GB | 28 GB | ~235 GB |
| Samsung EVO 870 | USB SSD | 1 TB | — | External |

---

## 🆕 ALphaMAIN — Personal Gaming PC

| Spec | Detail |
|------|--------|
| **Vendor** | ASUSTek Computer Inc. (OUI `30:C5:99`) |
| **MAC Address** | `30:c5:99:ef:5c:b4` |
| **IP Address** | `10.0.1.233` (DHCP — was `10.0.1.141`; lease moved ~Aug 20 2026, verified via ARP Aug 23) |
| **First Seen** | ~Aug 9 2026 (DHCP lease renewed Aug 9 ~08:00 EDT) |
| **OS** | Windows |
| **Status** | 🟡 Personal machine — usually off; SSH-monitor only |
| **Policy** | **NO agents, NO services, NO Hermes installs.** Joseph's personal gaming PC. SSH key access (`id_alphamain` on alphapi5) for connectivity checks + on-request help only. |
| **Wake-on-LAN** | Planned (Joseph to enable in BIOS + Windows). Pi has `wakeonlan` ready — magic packet to `30:c5:99:ef:5c:b4`. LAN-only, no WAN forward (security note). |

> **Added 2026-08-09** — was invisible to all monitoring (checks only probed hardcoded hosts). Tracked in [CHANGELOG.md](CHANGELOG.md). **IP moved to .233 Aug 2026** — docs updated 2026-08-23.

---

## 🆕 ALpha-Server — Frigate NVR / GPU Box

| Spec | Detail |
|------|--------|
| **Vendor** | ASUSTek Computer Inc. (OUI `D4:5D:64`) — ASUS board NIC |
| **MAC Address** | `d4:5d:64:aa:45:b1` |
| **IP Address** | `10.0.1.135` (DHCP, enp7s0) |
| **Hostname** | `ALpha-Server` (SSH alias `alpha-server` on alphapi5, user `alphaserver`) |
| **OS** | Ubuntu 24.04 x86_64, kernel 7.0.0-29-generic (built Aug 12 2026) |
| **CPU** | AMD Ryzen 5 3600 (6 cores, 12 threads) |
| **RAM** | 16 GB (15 GiB usable) |
| **Storage** | ~256 GB SSD (`/dev/sdb2`, 27 GB used / 211 GB free) |
| **GPU** | AMD RX 580 (VAAPI hardware acceleration for Frigate) |
| **Tailscale** | `100.123.100.38` |
| **Docker** | Frigate NVR (`ghcr.io/blakeblackshear/frigate:stable`, healthy, `restart=unless-stopped`) |
| **First Seen** | Home dir created ~Aug 1 2026; SSH alias on alphapi5 dated Aug 2 2026; kernel updated Aug 12; up since ~Aug 20 2026 |
| **Status** | ✅ Online — up 2+ days |

### Role

- **Frigate NVR** — AI object detection on the Eufy camera streams. Consumes RTSP from the Pi 5 eufy bridge (`rtsp://10.0.1.100:8554/...`); cameras include backyard, doorbell, and more (config in `/home/alphaserver/services/frigate/config/config.yml`).
- **Detectors:** CPU (3 threads) + VAAPI on the RX 580.
- **Storage:** `/home/alphaserver/media/frigate/storage` → `/media/frigate` (14-day alert/detection retention).
- Also hosts Hermes + Claude Code and the `lan-scan/` tooling; `services/` dir has configs for baby-tracker, immich, jellyfin, n8n (not all running yet — only Frigate is live as of 2026-08-23).

> **Added 2026-08-23** — set up ~Aug 1-4 but **never documented**: the Aug 9 LAN investigation knew the IP (it sat in the weekly check's KNOWN_HOSTS) yet it never reached HARDWARE.md/NETWORK.md. Found 2026-08-23 via ARP/MAC scan after the weekly lease fetch failed. Tracked in [CHANGELOG.md](CHANGELOG.md).

---

## 🖥️ alphamox — Mac Mini Proxmox

| Spec | Detail |
|------|--------|
| **Model** | Mac Mini (Late 2014) — A1347 |
| **CPU** | Intel Core i5-4260U @ 1.4 GHz (2 cores, 4 threads) |
| **RAM** | 8 GB DDR3 (5.1 GB used, 2.6 GB available) |
| **Storage** | 256 GB Apple proprietary SSD |
| **OS** | Proxmox VE 9.1.1 |
| **Hostname** | `alphamox` |
| **IP Address** | `10.0.1.108` |
| **MAC Address** | N/A |
| **Uptime** | 57+ days |
| **Form Factor** | Headless — no display, no peripherals |

### Virtual Machines & Containers

| Name | Type | RAM | Disk | Notes |
|------|------|-----|------|-------|
| Home Assistant | VM | 4 GB | 32 GB | Primary home automation |
| Omada Controller | LXC | — | — | TP-Link SDN controller |


## 💻 alphamobile-1 — Dell Latitude 5501

| Spec | Detail |
|------|--------|
| **Model** | Dell Latitude 5501 |
| **CPU** | Intel Core i7-9850H (6 cores, 12 threads @ up to 4.6 GHz) |
| **RAM** | ~7.5 GB |
| **Storage** | 238 GB NVMe SSD |
| **OS** | Ubuntu 24.04 LTS |
| **Hostname** | `alphamobile-1` |
| **IP Address** | `10.0.1.176` (LAN) · `100.82.167.20` (Tailscale) |
| **Status** | 🔴 **OFFLINE** — left the network ~2026-08-20 03:25 EDT, has not returned (17 days as of 2026-09-06) |
| **Uptime** | Was up since migration (~May 2026); unreachable since 2026-08-20 |
| **Form Factor** | Repurposed laptop (lid closed, headless) |

### Role

The Latitude is the **heavy lifter** — it runs CPU/IO-intensive services that the Pi 5's ARM architecture struggles with:

> **⚠️ 2026-09-06: The Latitude is currently OFFLINE** (not on the network since ~2026-08-20 03:25 EDT). All services it hosted are therefore **down**: Jellyfin, Immich, PostgreSQL + Redis, n8n, FreshRSS, RomM, Homepage, Dozzle, Watchtower. Detection trail: Uptime Kuma went EHOSTUNREACH at 2026-08-20 03:25 (after being green Aug 5–19), Tailscale shows `alphamobile-1` offline (last seen 17d ago), and the 2026-09-06 ARP sweep gets no reply. Likely related to ALpha-Server being stood up ~Aug 20 — Joseph may intend ALpha-Server to take over these roles, but as of 2026-09-06 ALpha-Server runs **only Frigate** (its `services/` dir has immich/jellyfin/n8n compose configs that are NOT running). Action: power the Latitude back on, or finish migrating its services to ALpha-Server.

- **Jellyfin** — hardware-accelerated video transcoding (Intel Quick Sync)
- **Immich** — photo ML (facial recognition, object detection)
- **PostgreSQL + Redis** — database backend for services
- **n8n** — workflow automation
- **FreshRSS, RomM, Homepage, Dozzle, Watchtower**

### Migration Story

Originally everything ran on the Pi 5. As the homelab grew, three bottlenecks became clear:

| Bottleneck | Why It Mattered | How the Latitude Fixed It |
|------------|----------------|---------------------------|
| **ARM transcoding** | Jellyfin had to software-transcode every video — 100% CPU on a single stream | x86 + Quick Sync = hardware-accelerated, near-zero CPU |
| **RAM pressure** | Immich ML + Jellyfin + *arr stack left only 3.5 GB free | 7.5 GB available, plenty of headroom |
| **SD card writes** | Database services constantly write to disk — SD cards die fast | NVMe SSD, designed for sustained I/O |

The migration moved all heavyweight services to the Latitude (now accessed at `10.0.1.176`), leaving the Pi 5 to handle lightweight orchestration — *arr stack, Zigbee coordinator, Pi-hole, RDTClient, Eufy bridge.

### Cross-Host Routing Note

Due to a switch-level quirk, the Pi 5 cannot reach the Latitude on the local subnet directly. All cross-host traffic routes through Tailscale (`100.82.167.20`). See [hardware/latitude.md](hardware/latitude.md) for details.

---

## 🌐 OPNsense — Dell OptiPlex Router/Firewall

| Spec | Detail |
|------|--------|
| **Model** | Dell OptiPlex (small form factor) |
| **OS** | OPNsense |
| **Role** | Router / Firewall / DHCP server |
| **IP Address** | `10.0.1.1` |
| **Uptime** | 57+ days |

### Network Topology

```
ISP ONT (Fiber modem)
    │
    ▼
OPNsense (Dell OptiPlex) — WAN
    │
    ├── LAN (10.0.1.0/24)
    │   ├── ALphaMAIN (10.0.1.233) — NEW Aug 2026 (was .141)
    │   ├── ALpha-Server (10.0.1.135) — NEW Aug 2026 (Frigate NVR)
    │   ├── alphamobile-1 (Latitude 5501 — 10.0.1.176) 🔴 OFFLINE since 2026-08-20
    │   ├── alphapi5 (10.0.1.100)
    │   ├── alphamox (10.0.1.108)
    │   ├── alphapi3 (10.0.1.158)
    │   ├── EAP 670 (WiFi 6 AP)
    │   ├── HP Printer (10.0.1.103)
    │   ├── HP Printer (10.0.1.116)
    │   └── Eufy HomeBase 2 (10.0.1.187)
    │
    ├── VLAN 1 — Trusted (10.0.10.0/24)
    ├── VLAN 2 — Services (10.0.20.0/24)
    ├── VLAN 3 — IoT (10.0.30.0/24)
    └── VLAN 4 — Guest (10.0.40.0/24)
```

### DHCP Scope

| Parameter | Value |
|-----------|-------|
| **Subnet** | `10.0.1.0/24` |
| **Gateway** | `10.0.1.1` |
| **DHCP Server** | OPNsense |

---

## 🧊 alphapi3 — Raspberry Pi 3 (Back Online)

| Spec | Detail |
|------|--------|
| **Model** | Raspberry Pi 3 Model B+ Rev 1.3 |
| **RAM** | 1 GB |
| **CPU** | 4× ARM Cortex-A53 @ 1.4 GHz |
| **Kernel** | — (Ubuntu 24.04) |
| **OS** | Ubuntu 24.04 |
| **IP Address** | `10.0.1.158` (was `10.0.1.101`) |
| **MAC Address** | `B8:27:EB:CF:91:C8` |
| **Status** | ✅ Online — up 6+ weeks |
| **Last Temperature** | ~48 °C idle |

### Storage

| Device | Type | Capacity | Used | Notes |
|--------|------|----------|------|-------|
| SD Card | microSD | 128 GB | 3.4 GB | — |

### Status

- Previously used as a **K3s worker node** (no pods scheduled when last online).
- **Planned repurpose**: LibreELEC travel media stick.

---

## 🌍 Network Hardware

### TP-Link TL-SG108E

| Spec | Detail |
|------|--------|
| **Type** | 8-port Gigabit managed switch |
| **Ports** | 8× Gigabit Ethernet |
| **Backbone** | 1 Gbps full duplex |
| **Uplinks** | OPNsense (LAN) → switch → all endpoints |

### TP-Link EAP 670

| Spec | Detail |
|------|--------|
| **Type** | WiFi 6 (802.11ax) Access Point |
| **Standard** | WiFi 6 |
| **Management** | Omada SDN Controller (alphamox LXC) |

### ISP ONT

| Spec | Detail |
|------|--------|
| **Type** | Fiber ONT (ISP-provided) |
| **Connection** | Fiber → Ethernet to OPNsense WAN |

---

## 🖨️ Printers

| IP Address | Notes |
|-----------|-------|
| `10.0.1.103` | HP Printer |
| `10.0.1.116` | HP Printer |

---

## 📡 Zigbee Coordinator

| Spec | Detail |
|------|--------|
| **Device** | Sonoff Zigbee 3.0 USB Dongle Plus V2 (ZBDongle-E) |
| **Firmware** | EmberZNet 7.4.5 |
| **Connected To** | alphapi5 (Raspberry Pi 5) — USB `/dev/ttyUSB0` |
| **Software** | zigbee2mqtt (Docker) |
| **Network** | PAN 34179, channel 11 |
| **Paired Devices** | 8 (1 Aqara WSDCGQ11LM temp/humidity, 2 Third Reality 3RMS16BZ motion, 4 Third Reality 3RCB01057Z Smart Color Bulb, 1 eWeLink CK-TLSR8656) |
| **Pending** | 3 motion sensors, 1 humidity, 2 door sensors |

---

## ⚡ Power & Environment

- All online nodes have been **up for 57+ days** (since last maintenance) — except ALpha-Server, which joined ~Aug 20 2026.
- **alphamobile-1 (Latitude) has been OFFLINE since ~2026-08-20** — see its section above. Its heavy services (Jellyfin, Immich, n8n, FreshRSS, RomM, etc.) are down with it; ALpha-Server (Frigate only) has not yet taken them over.
- alphapi5 idles at ~61 °C (under typical passive cooling).
- alphapi3 is back online at `10.0.1.158` (was thought powered off — see status above).
- No UPS currently documented.
