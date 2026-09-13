# Raspberry Pi 5 — Homelab Server

## Overview

The Raspberry Pi 5 serves as the lightweight orchestrator for the homelab, running containerized services including media automation, network security (Pi-hole), home automation bridging, Zigbee coordination **and (since 2026-09-08) the Immich photo stack**. Services are managed via Docker Compose — the Pi 5 now carries 19 containers on its own, with Frigate on ALpha-Server.

---

## Role Change: The Migration

Originally the Pi 5 ran everything. In mid-2026, CPU/IO-heavy services (Jellyfin, Immich, n8n, RomM, FreshRSS) migrated to a Dell Latitude 5501 (i7-9850H, x86) to overcome three bottlenecks:

| Bottleneck | How the Latitude Fixed It |
|------------|---------------------------|
| **ARM software transcoding** — 100% CPU on a single Jellyfin stream | Intel Quick Sync = hardware-accelerated, near-zero CPU |
| **RAM pressure** — Immich ML + Jellyfin + *arr left only 3.5 GB free | 7.5 GB available on the Latitude |
| **SD card writes** — database services wear out SD cards fast | NVMe SSD on the Latitude |

The Pi 5 now handles **lightweight orchestration**: *arr stack, Zigbee coordinator, Pi-hole, RDTClient, Eufy bridge, FlareSolverr, Uptime Kuma, Ntfy, Watchtower, go2rtc — **plus Immich again since 2026-09-08** (migrated back after the Latitude went offline). See [latitude.md](latitude.md) for the heavy lifter's history.

---

## Hardware Specifications

| Component | Detail |
|---|---|
| **Model** | Raspberry Pi 5 |
| **RAM** | 8 GB LPDDR4X |
| **Boot Storage** | 256 GB microSD card |
| **Secondary Storage** | 1 TB Samsung EVO 870 SSD (USB 3.0 attached) |
| **CPU** | Quad-core ARM Cortex-A76 @ 2.4 GHz |
| **Idle Temperature** | ~61°C |
| **MAC Address** | `2c:cf:67:27:49:1a` |

---

## Software Stack

| Layer | Technology |
|---|---|
| **Operating System** | Ubuntu 24.04 LTS (Noble Numbat) |
| **Container Runtime** | Docker + Docker Compose (per-service files) |
| **Orchestration** | K3s (lightweight Kubernetes) with Flannel CNI |
| **VPN / Mesh** | Tailscale |
| **Firewall** | UFW |

---

## Network Configuration

| Parameter | Value |
|---|---|
| **Hostname** | `pi5` (or configured hostname) |
| **Static IP** | `10.0.1.100/24` |
| **Gateway** | `10.0.1.1` |
| **Subnet** | `10.0.1.0/24` |
| **MAC Address** | `2c:cf:67:27:49:1a` |

### macvlan (Pi-hole)

A macvlan interface is configured to give the Pi-hole container its own IP on the local LAN:

- **Pi-hole IP:** `10.0.1.253/24`
- This allows Pi-hole to serve as the network-wide DNS resolver independent of the host's IP stack.

### Tailscale

- **Tailscale IP:** `100.101.94.73`
- Tailscale provides secure mesh VPN access to the Pi from outside the local network.
- **Important routing fix:** `--accept-routes=false` is set to prevent Tailscale from overriding local subnet routes, which would interfere with LAN traffic and Pi-hole DNS resolution.

### UFW Firewall Rules

UFW is configured to allow only essential traffic:

```text
Status: active

To                         Action      From
--                         ------      ----
22/tcp (SSH)               ALLOW       10.0.1.0/24
80/tcp (HTTP)              ALLOW       10.0.1.0/24
443/tcp (HTTPS)            ALLOW       10.0.1.0/24
9091/tcp (Transmission)    ALLOW       10.0.1.0/24
3000/tcp (Grafana/metrics) ALLOW       10.0.1.0/24
51820/udp (WireGuard)      ALLOW       Anywhere
6443/tcp (K3s API)         ALLOW       10.0.1.0/24
```

Default policies: incoming **deny**, outgoing **allow**.

---

## System Uptime & Performance

- **Uptime:** 57+ days (as of last reset — typically runs for months without interruption)
- **Idle Temperature:** ~61°C (ambient ~22°C)
- **CPU Load:** Typically low (< 1.0) at idle; moderate under media transcode load
- **Power Consumption:** ~7–15 W depending on workload

---

## Running Services (Docker Containers)

> **Since the migration**, Jellyfin, RomM, n8n, FreshRSS, Homepage, and Dozzle ran on the [Dell Latitude 5501](latitude.md). The Pi 5 kept the lightweight orchestration layer.

> **⚠️ 2026-09-13:** the Latitude has been **OFFLINE since ~2026-08-20**, so Jellyfin, RomM, n8n, FreshRSS, Homepage and Dozzle are **still DOWN**. **Immich, however, has been migrated BACK to the Pi 5** (2026-09-08) and is running healthy again — the four containers below are part of the Pi 5's 19-container fleet. See [latitude.md](latitude.md) / [CHANGELOG.md](../CHANGELOG.md).

### Media Automation (arr-suite)

| Service | Purpose |
|---|---|
| **sonarr** | TV series automatic downloader |
| **radarr** | Movie automatic downloader |
| **bazarr** | Subtitle management (automatic download & sync) |
| **prowlarr** | Indexer manager (feeds sonarr/radarr) |
| **rdtclient** | Real-Debrid torrent client bridge |
| **flaresolverr** | Cloudflare challenge solver for indexers |

### Network & DNS

| Service | Purpose |
|---|---|
| **pihole** | Network-wide ad blocking & DNS sinkhole |

### Smart Home Bridge

| Service | Purpose |
|---|---|
| **eufy-security-ws** | Eufy camera → Home Assistant bridge (port 3002) |
| **zigbee2mqtt** | Zigbee coordinator → MQTT bridge (port 8080) |
| **mosquitto** | MQTT message broker |
| **go2rtc** | RTSP restreamer — Eufy/camera streams to Home Assistant + Frigate (`rtsp://10.0.1.100:8554`, API origin = HA VM `10.0.1.154:8123`) |

### Photos

| Service | Purpose |
|---|---|
| **immich_server** | Immich photo/video library + web UI (`:2283`) — migrated back from the Latitude 2026-09-08 |
| **immich_machine_learning** | Immich ML — facial recognition, object/scene detection |
| **immich_postgres** | Immich PostgreSQL database (VectorChord/pgvecto.rs image) |
| **immich_redis** | Cache / job queue (Valkey 9) |

> Immich data lives on the NAS: library `UPLOAD_LOCATION=/mnt/nas-data/immich/library`, DB at `/mnt/nas-data/immich/postgres`. The family archive at `/mnt/nas-data/vault` is mounted **read-only** into `immich_server` as an external library (`/vault:ro`). Watchtower is opted out for Immich containers — upgrade manually (DB migrations).

### Monitoring

| Service | Purpose |
|---|---|
| **uptime-kuma** | Uptime monitoring dashboard with notifications |

### Management

| Service | Purpose |
|---|---|
| **portainer** | Docker management UI |
| **watchtower** | Auto-update Docker containers |

### Notifications

| Service | Purpose |
|---|---|
| **ntfy** | Push notification server (mobile push via ntfy.sh)

---

## Storage Layout

### Mount Points

```text
/                          → 256 GB microSD (OS + Docker configs + compose files)
/mnt/ssd                   → 1 TB Samsung EVO 870 SSD (media, databases, app data)
```

### Partitioning (SSD)

```
/dev/sda1   ext4   1 TB   /mnt/ssd   (data partition)
```

### Docker Volume Locations

- Compose files: `~/docker/<service>/compose.yml`
- Persistent data volumes are mapped to directories on the SSD at `/mnt/ssd/docker/<service>/`

### Swap Configuration

- **Swap:** Disabled / not configured
- Rationale: The 8 GB RAM is sufficient for the current workload; an SD-card swap file would cause excessive wear and degrade performance. If swap is needed in the future, it should be placed on the USB SSD.

---

## K3s (Kubernetes)

A lightweight Kubernetes cluster runs on the Pi using K3s with Flannel as the CNI plugin.

| Component | Detail |
|---|---|
| **Cluster Mode** | Single-node (server only) |
| **CNI** | Flannel (VXLAN backend) |
| **API Server** | `10.0.1.100:6443` |
| **Pod Network** | `10.42.0.0/16` |
| **Service Network** | `10.43.0.0/16` |

### Macvlan for Pi-hole (Kubernetes/Networking)

The Pi-hole container runs outside K3s via Docker Compose on a macvlan network, giving it the dedicated IP `10.0.1.253` on the LAN. This bypasses the host network stack for DNS traffic.

---

## SMB File Sharing

Samba is configured to share the SSD storage over the local network.

- **Share name:** `ssd` (or configured share name)
- **Path:** `/mnt/ssd`
- **Access:** Local LAN only (`10.0.1.0/24`)
- **Authentication:** User-level (credentials required)
- **Purpose:** Direct access to media files from workstations/laptops without going through Jellyfin

---

## Maintenance Tips

### Backups

- Back up Docker Compose files and config directories regularly (`~/docker/`).
- Immich database backups should be taken before OS or Immich version upgrades.
- Pi-hole Teleporter export recommended on a schedule.

### Temperature

- If temperatures exceed 75°C under sustained load, consider additional heatsinking or a small fan.
- Idle temp of ~61°C is normal for the Pi 5 in a passively cooled case.

### SD Card Lifespan

- The SD card serves only the OS and configuration files. Heavy I/O (media, databases, downloads) is directed to the USB SSD.
- Consider using log2ram to reduce SD card writes.

### Updates

```bash
# System packages
sudo apt update && sudo apt upgrade -y

# Docker images (pull latest and recreate)
cd ~/docker/<service>
docker compose pull && docker compose up -d

# K3s updates
sudo k3s upgrade
```

### Monitoring Checks

- **Disk usage:** `df -h` — watch the SD card root partition; logs can fill it unexpectedly.
- **Container health:** `docker ps` or via Portainer UI.
- **Pi-hole logs:** Rotate or truncate `pihole.log` to prevent disk exhaustion.
- **Tailscale status:** `tailscale status` to verify mesh connectivity.

---

## Quick Reference

| Command | Purpose |
|---|---|
| `ssh alpha@10.0.1.100` | SSH into the Pi over LAN |
| `docker compose up -d` | Start/update a service stack |
| `docker compose logs -f` | Follow logs for a service |
| `tailscale ping 100.101.94.73` | Test Tailscale connectivity |
| `sudo ufw status numbered` | Check firewall rules |
| `k3s kubectl get nodes` | Check K3s cluster status |
| `sudo smbstatus` | Check active SMB connections |

---

## Notes

- All Docker containers run on the host network or macvlan (Pi-hole) — no overlay Docker networks are used except for inter-container communication within a compose stack.
- The Pi's 8 GB RAM is sufficient for all listed services simultaneously, though memory pressure can build during heavy Immich ML processing. Monitor with `htop` or `docker stats`.
- Tailscale `--accept-routes=false` is critical — without it, Tailscale's subnet routing competes with the local LAN routes and breaks local DNS/macvlan access.
