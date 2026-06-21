# 🏠 Alpha Network Homelab

> **Infrastructure tinkering. Smart home automation. Media streaming. All from a Pi 5, a salvaged Mac Mini, and an old OptiPlex.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

This repo documents my home lab — what it runs, how it's wired, and why I built it that way. It's part reference for myself, part portfolio piece, and part "hey friends, here's what I've been nerding out on."

---

## 🧠 Philosophy

- **Use what you have.** Repurpose old hardware before buying new.
- **Cost-conscious but capable.** Free and open-source tools first, subscriptions only where they earn their keep.
- **Privacy-first.** Self-hosted where it counts (DNS, cameras, file storage, voice control).
- **Documented as built.** If it's not written down, it doesn't exist.

---

## 📦 At a Glance

| System | Role | Platform | OS |
|--------|------|----------|----|
| **alphapi5** | Primary server | Raspberry Pi 5 (8GB) | Ubuntu 24.04 |
| **alphamox** | Hypervisor | Mac Mini (Late 2014) | Proxmox VE 9.1 |
| **alphapi3** | Secondary node / travel stick | Raspberry Pi 3 B+ | Ubuntu 24.04 |
| **OPNsense** | Router/firewall | Dell OptiPlex | OPNsense |

**20+ Docker containers** | **3-person Tailscale mesh** | **Home Assistant** | **50+ smart home entities**

---

## 🖥️ What It Does

| Category | Services |
|----------|----------|
| **🎬 Media** | Jellyfin, Sonarr, Radarr, Bazarr, Prowlarr, RDTClient, FlareSolverr |
| **📸 Photos** | Immich (Google Photos alternative) |
| **🎮 ROMs** | RomM (game ROM manager) |
| **🏠 Smart Home** | Home Assistant, Eufy cameras, LG TV, lights (Tuya/Kasa), Alexa voice |
| **🌐 Network** | Pi-hole (DNS), Omada (WiFi management), Tailscale (mesh VPN) |
| **🤖 Automation** | n8n, cron-based housekeeping |
| **📡 Monitoring** | Uptime Kuma, custom health check scripts |
| **📰 Content** | FreshRSS (RSS reader) |
| **🔔 Notifications** | Ntfy (push notifications) |
| **🐳 Management** | Portainer (Docker UI) |

---

## 🗺️ Network Topology

```
Internet
   │
   ▼
[ONT] ─── [OPNsense Router] ─── WAN subnet
               │
               ▼
         [TL-SG108E Switch]
          /     │      \
         ▼      ▼      ▼
      Pi 5   Mac Mini  EAP 670
   (Server)  (Proxmox)  (WiFi)
      │
      ▼
   Pi 3 (secondary)
```

> *A proper SVG diagram is in `images/topology.svg`.*

---

## 🔗 Quick Links

| Document | What's In It |
|----------|-------------|
| [HARDWARE.md](HARDWARE.md) | Every physical thing with specs |
| [NETWORK.md](NETWORK.md) | IP scheme, VLANs, firewall rules |
| [hardware/pi5.md](hardware/pi5.md) | Pi 5 — the workhorse |
| [hardware/mac-mini.md](hardware/mac-mini.md) | Mac Mini Proxmox setup |
| [hardware/optiplex.md](hardware/optiplex.md) | OPNsense router |
| [hardware/pi3.md](hardware/pi3.md) | Pi 3 — retired & travel plans |
| [services/media-stack.md](services/media-stack.md) | Jellyfin + *arr automation |
| [services/smart-home.md](services/smart-home.md) | HA, cameras, lights, voice |
| [services/network-services.md](services/network-services.md) | DNS, WiFi, mesh VPN |
| [services/automation.md](services/automation.md) | n8n, cron, schedules |
| [services/storage.md](services/storage.md) | Photos, ROMs, file sharing |
| [services/monitoring.md](services/monitoring.md) | Uptime, health checks |
| [docs/remote-access.md](docs/remote-access.md) | Tailscale & VPN setup |
| [docs/security.md](docs/security.md) | Firewalls, access control, secrets |

---

## 🧰 Lessons Learned

A few things I'd tell myself if I were starting over:

- **Start with the network.** OPNsense + a managed switch gives you VLANs from day one, and VLANs make everything cleaner later.
- **Document as you go.** This repo exists because I didn't, and reconstructing everything from memory is painful.
- **One service, one container.** Don't cram things together — Docker compose per service makes debugging 10x easier.
- **Tailscale is magic, but `--accept-routes` is a trap.** When a Tailscale node advertises subnet routes, other nodes accept them by default — routing local LAN traffic through the mesh tunnel and breaking direct connectivity. Set `--accept-routes=false` on non-router nodes.
- **SD cards are temporary.** Log everything properly or your Pi's SD card will die faster than expected.
- **Don't let NBD stay connected.** Running a QEMU NBD mount while the VM is running will corrupt the disk. Always disconnect before starting the VM.

---

## 📄 License

MIT — use it, share it, build on it.

---

*Built with ❤️ and far too many late nights.*
