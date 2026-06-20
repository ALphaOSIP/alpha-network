# Raspberry Pi 5 — Homelab Server

## Overview

The Raspberry Pi 5 serves as the primary homelab server, running a diverse stack of containerized services including media streaming, photo management, network security (Pi-hole), home automation, and a lightweight Kubernetes (K3s) cluster. All services are managed via Docker Compose with persistent storage on both SD card and a USB-attached SSD.

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

All services run as individual Docker Compose stacks, each in their own directory with a dedicated `compose.yml` (or `docker-compose.yml`) file.

### Media Stack (arr-suite + Jellyfin)

| Service | Purpose |
|---|---|
| **jellyfin** | Media server (movies, TV, music) |
| **sonarr** | TV series automatic downloader |
| **radarr** | Movie automatic downloader |
| **bazarr** | Subtitle management (automatic download & sync) |
| **prowlarr** | Indexer manager (feeds sonarr/radarr) |
| **rdtclient** | Real-Debrid torrent client bridge |
| **flaresolverr** | Cloudflare challenge solver for indexers |

### Photo Management

| Service | Purpose |
|---|---|
| **immich-server** | Self-hosted Google Photos alternative |
| **immich-postgres** | PostgreSQL database for Immich |
| **immich-redis** | Redis cache for Immich |
| **immich-machine-learning** | ML-based image classification & face recognition |

### Gaming

| Service | Purpose |
|---|---|
| **romm** | ROM manager for retro game emulation |
| **romm-db** | MariaDB database for RomM |

### Network & DNS

| Service | Purpose |
|---|---|
| **pihole** | Network-wide ad blocking & DNS sinkhole |

### Automation

| Service | Purpose |
|---|---|
| **n8n** | Workflow automation (Zapier-like, self-hosted) |

### Monitoring

| Service | Purpose |
|---|---|
| **uptime-kuma** | Uptime monitoring dashboard with notifications |

### Management

| Service | Purpose |
|---|---|
| **portainer** | Docker management UI |

### Notifications

| Service | Purpose |
|---|---|
| **ntfy** | Push notification server (mobile push via ntfy.sh or self-hosted) |

### RSS

| Service | Purpose |
|---|---|
| **freshrss** | RSS/Atom feed aggregator |

### Smart Home

| Service | Purpose |
|---|---|
| **eufy-security-ws** | WebSocket bridge for Eufy security cameras/devices |

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
