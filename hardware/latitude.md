# Dell Latitude 5501 — Heavy Lifter

## Overview

The Latitude 5501 started as a retired work laptop, now repurposed as the homelab's heavyweight server. It handles CPU-intensive services that the Pi 5's ARM chip struggles with — media transcoding, photo ML processing, and database workloads.

Hostname: `alphamobile-1`

> "Use what you have." — This machine cost $0 (already owned) and doubled the lab's compute capacity overnight.

---

## Hardware Specifications

| Component | Detail |
|---|---|
| **Model** | Dell Latitude 5501 |
| **CPU** | Intel Core i7-9850H (6 cores, 12 threads @ up to 4.6 GHz) |
| **RAM** | ~7.5 GB (of total capacity) |
| **Storage** | 238 GB NVMe SSD |
| **OS** | Ubuntu 24.04 LTS |
| **Network** | Gigabit Ethernet (eth0) — `10.0.1.176` |
| **Tailscale** | `100.82.167.20` |
| **Form Factor** | Laptop chassis (lid closed, headless operation) |

---

## Why the Migration

The original setup ran everything on the Pi 5. As the lab grew, the Pi 5 started showing strain:

| Problem | Impact |
|---|---|
| **ARM architecture** | Jellyfin had to software-transcode video — pegged all 4 cores at 100% for a single 1080p stream |
| **Memory pressure** | Immich's ML pipeline (facial recognition, object detection) consumed 2-3 GB of RAM, leaving only ~3.5 GB for everything else |
| **SD card wear** | Database-heavy services (Immich postgres, n8n) hammer the storage with constant writes — bad for SD card longevity |

The Latitude solved all of this in one shot:
- **x86 CPU with Quick Sync** — hardware-accelerated transcoding, almost zero CPU hit
- **7.5 GB available RAM** — runs all heavy services with room to spare
- **NVMe SSD** — way faster I/O than SD or USB for database workloads

---

## Services Running

The Latitude hosts the heavyweight services that *need* x86 power:

| Service | Port | Purpose |
|---|---|---|
| **Jellyfin** | `:8096` | Media streaming & hardware-accelerated transcoding |
| **Immich** | `:2283` | Google Photos alternative with ML tagging/faces |
| **RomM** | `:3000` | Game ROM library manager |
| **n8n** | `:5678` | Workflow automation engine |
| **FreshRSS** | `:8082` | RSS feed reader |
| **Homepage** | `:3001` | Custom service dashboard |
| **Dozzle** | `:8888` | Docker log viewer |
| **Watchtower** | — | Auto-update Docker containers |
| **PostgreSQL** | — | Database backend for Immich & other services |
| **Redis** | — | Caching layer for various services |

The Pi 5 now handles only the lightweight orchestration layer — *arr stack, Zigbee, Pi-hole, RDTClient, Eufy bridge.

---

## Network Note

Due to a switch-level quirk, the Pi 5 (`10.0.1.100`) cannot directly reach the Latitude on the local subnet. All cross-host communication routes through Tailscale instead — the Latitude is accessed at `100.82.167.20` from the Pi 5 side. This doesn't affect normal operation (all Docker services bind to their ports), but any host-to-host scripts or monitoring use the Tailscale IP.

---

## Quick Reference

| Command | Purpose |
|---|---|
| `ssh alpha@100.82.167.20` | SSH via Tailscale (preferred) |
| `ssh alpha@10.0.1.176` | SSH over LAN (only works from same switch port) |
| `docker compose up -d` | Start/update a service |
| `tailscale ping 100.82.167.20` | Test connectivity from Pi 5 |
