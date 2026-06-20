# 📋 Services Overview

A categorized index of every service running in the Alpha Network homelab.

---

## 🎬 Media & Entertainment

| Service | URL | Purpose |
|---------|-----|---------|
| **Jellyfin** | `http://10.0.1.100:8096` | Media streaming (movies, TV, music) |
| **Sonarr** | `http://10.0.1.100:8989` | TV show automation |
| **Radarr** | `http://10.0.1.100:7878` | Movie automation |
| **Bazarr** | `http://10.0.1.100:6767` | Subtitle management |
| **Prowlarr** | `http://10.0.1.100:9696` | Indexer manager |
| **RDTClient** | `http://10.0.1.100:6500` | Real-Debrid download client |
| **FlareSolverr** | `http://10.0.1.100:8191` | Cloudflare bypass proxy |

> *See [media-stack.md](media-stack.md) for the full architecture.*

---

## 📸 Photos & Memory

| Service | URL | Purpose |
|---------|-----|---------|
| **Immich** | `http://10.0.1.100:2283` | Google Photos alternative, ML-powered |

---

## 🎮 Gaming

| Service | URL | Purpose |
|---------|-----|---------|
| **RomM** | `http://10.0.1.100:3000` | Game ROM manager & library |

---

## 🏠 Smart Home

| Service | URL/Entity | Purpose |
|---------|------------|---------|
| **Home Assistant** | `http://10.0.1.154:8123` | Smart home hub |
| **Eufy Security** | Via HA | 5 security cameras, doorbell |
| **Gosund Bulbs** | `light.headlight`, `light.headlight_2` | Tuya smart bulbs |
| **TP-Link KL130** | `light.bedroom_lamp` | Matter-compatible RGB bulb |
| **LG TV** | `media_player.lg_webos_tv_55ur8000aua` | Smart TV control |
| **Alexa Media Player** | Via HA | Voice control integration |

> *See [smart-home.md](smart-home.md) for integration details.*

---

## 🌐 Network & DNS

| Service | URL | Purpose |
|---------|-----|---------|
| **Pi-hole** | `http://10.0.1.253/admin` | DNS sinkhole, ad blocking |
| **Omada Controller** | `http://10.0.1.141:8088` | WiFi management (EAP 670) |
| **Tailscale** | Mesh VPN | Encrypted remote access |

> *See [network-services.md](network-services.md) for setup details.*

---

## 🤖 Automation

| Service | URL | Purpose |
|---------|-----|---------|
| **n8n** | `http://10.0.1.100:5678` | Workflow automation |
| **Goodnight House** | Cron (Mac Mini) | 2am lights/TV shutdown |
| **DeepSeek Monitor** | Cron (Pi 5) | Daily API balance check |
| **Network Monitor** | Cron (Pi 5) | Service health checks |

> *See [automation.md](automation.md) for workflow details.*

---

## 📡 Monitoring & Management

| Service | URL | Purpose |
|---------|-----|---------|
| **Uptime Kuma** | `http://10.0.1.100:3001` | Service uptime monitoring |
| **Ntfy** | `http://10.0.1.100:2586` | Push notifications |
| **Portainer** | `http://10.0.1.100:9000` | Docker container management |

> *See [monitoring.md](monitoring.md) for setup details.*

---

## 📰 Content & Notifications

| Service | URL | Purpose |
|---------|-----|---------|
| **FreshRSS** | `http://10.0.1.100:8082` | RSS feed reader |

---

## 🗄️ Storage

| Service | Path | Purpose |
|---------|------|---------|
| **Samba Share** | `\\\\10.0.1.100\\shared` | Network file sharing |
| **ROM Library** | `/mnt/nas-data/media/roms/` | Game ROM storage |

> *See [storage.md](storage.md) for storage architecture.*

---

## Quick Access

| What | Where |
|------|-------|
| **Docker host** | `ssh alpha@10.0.1.100` |
| **Proxmox** | `ssh root@10.0.1.108` |
| **HA CLI** | `ssh root@10.0.1.108` → `qm terminal 100` |
| **All services** | [Uptime Kuma](http://10.0.1.100:3001) |
