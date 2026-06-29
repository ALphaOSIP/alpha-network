# 🖥️ Alpha-Network Hardware Inventory

> Last updated: 2026-06-20

This document catalogs all hardware in the Alpha homelab network (10.0.1.0/24).

---

## 📋 Summary

| Device | Role | OS | IP | CPU | RAM | Status |
|--------|------|----|----|-----|-----|--------|
| **alphapi5** | General-purpose server | Ubuntu 24.04.1 LTS | — | 4× Cortex-A76 @ 2.4 GHz | 8 GB | ✅ Online |
| **alphamox** | Proxmox hypervisor | Proxmox VE 9.1.1 | 10.0.1.108 | i5-4260U (2C/4T) @ 1.4 GHz | 8 GB DDR3 | ✅ Online |
| **OPNsense (Dell OptiPlex)** | Router / Firewall | OPNsense | 10.0.1.1 | — | — | ✅ Online |
| **alphapi3** | K3s worker node (offline) | Ubuntu 24.04 | 10.0.1.101 | 4× Cortex-A53 @ 1.4 GHz | 1 GB | ❌ Offline (travel plans) |
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
    ▼
TL-SG108E (managed switch) — LAN
    ├── alphapi5 (eth0)
    ├── alphamox (10.0.1.108)
    ├── alphapi3 (10.0.1.101) — OFFLINE
    ├── EAP 670 (WiFi 6 AP)
    ├── HP Printer (10.0.1.103)
    └── HP Printer (10.0.1.116)
```

### DHCP Scope

| Parameter | Value |
|-----------|-------|
| **Subnet** | `10.0.1.0/24` |
| **Gateway** | `10.0.1.1` |
| **DHCP Server** | OPNsense |

---

## 🧊 alphapi3 — Raspberry Pi 3 (Offline)

| Spec | Detail |
|------|--------|
| **Model** | Raspberry Pi 3 Model B+ Rev 1.3 |
| **RAM** | 1 GB |
| **CPU** | 4× ARM Cortex-A53 @ 1.4 GHz |
| **Kernel** | — (Ubuntu 24.04) |
| **OS** | Ubuntu 24.04 |
| **IP Address** | `10.0.1.101` |
| **MAC Address** | `B8:27:EB:CF:91:C8` |
| **Status** | ❌ Powered off |
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
| **Paired Devices** | 4 (Aqara temp/humidity, Third Reality motion, 2 TRÅDFRI bulbs) |
| **Pending** | 3 motion sensors, 1 humidity, 2 door sensors |

---

## ⚡ Power & Environment

- All online nodes have been **up for 57+ days** (since last maintenance).
- alphapi5 idles at ~61 °C (under typical passive cooling).
- alphapi3 is powered off awaiting re-purpose.
- No UPS currently documented.
