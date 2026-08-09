# 🖥️ Alpha-Network Hardware Inventory

> Last updated: 2026-08-09

This document catalogs all hardware in the Alpha homelab network (10.0.1.0/24).

---

## 📋 Summary

| Device | Role | OS | IP | CPU | RAM | Status |
|--------|------|----|----|-----|-----|--------|
| **ALphaMAIN** | New main machine (ASUS) | TBD | 10.0.1.141 | TBD | TBD | 🟡 New — seen on DHCP, currently asleep |
| **alphamobile-1 (Latitude)** | Heavy lifter server | Ubuntu 24.04 LTS | 10.0.1.176 | i7-9850H (6C/12T) @ 4.6 GHz | ~7.5 GB | ✅ Online |
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

## 🆕 ALphaMAIN — New ASUS Machine

| Spec | Detail |
|------|--------|
| **Vendor** | ASUSTek Computer Inc. (OUI `30:C5:99`) |
| **MAC Address** | `30:c5:99:ef:5c:b4` |
| **IP Address** | `10.0.1.141` (DHCP — old Omada LXC IP, freed Aug 2026) |
| **First Seen** | ~Aug 9 2026 (DHCP lease renewed Aug 9 ~08:00 EDT) |
| **Status** | 🟡 New — asleep when probed (no ping/SSH/ports), role TBD |
| **Notable** | Follows ALpha* naming convention (ALphaMAIN). Likely Joseph's new main desktop/laptop (ASUS). SSH key not yet provisioned from alphapi5. |

> **Added 2026-08-09** — this machine was invisible to all monitoring (connectivity check, weekly doc cron) because those checks only probe hardcoded hosts and never scan the LAN. Tracked in [CHANGELOG.md](CHANGELOG.md).

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
| **Uptime** | Since migration (~May 2026) |
| **Form Factor** | Repurposed laptop (lid closed, headless) |

### Role

The Latitude is the **heavy lifter** — it runs CPU/IO-intensive services that the Pi 5's ARM architecture struggles with:

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
    │   ├── ALphaMAIN (10.0.1.141) — NEW Aug 2026
    │   ├── alphamobile-1 (Latitude 5501 — 10.0.1.176)
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

- All online nodes have been **up for 57+ days** (since last maintenance).
- alphapi5 idles at ~61 °C (under typical passive cooling).
- alphapi3 is powered off awaiting re-purpose.
- No UPS currently documented.
