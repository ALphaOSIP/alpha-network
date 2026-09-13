# Dell Latitude 5501 — Heavy Lifter

> **🔴 OFFLINE since ~2026-08-20 03:25 EDT** (verified again 2026-09-13 — Uptime Kuma EHOSTUNREACH, Tailscale last seen 2026-08-20, no ARP reply). **Immich has been migrated back to the Pi 5 (2026-09-08) and is healthy**; Jellyfin, RomM, n8n, FreshRSS, Homepage and Dozzle are **still DOWN** until this machine is powered back on or its remaining workload is migrated. See [CHANGELOG.md](../CHANGELOG.md).

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

The Latitude hosts the heavyweight services that *need* x86 power — **currently OFFLINE, so most of these are not running (as of 2026-09-13)**:

| Service | Port | Purpose |
|---|---|---|
| **Jellyfin** | `:8096` | Media streaming & hardware-accelerated transcoding — ⬇️ down |
| ~~**Immich**~~ | `:2283` | ⚠️ **Migrated back to the Pi 5 (`10.0.1.100:2283`) on 2026-09-08 — healthy again** |
| **RomM** | `:3000` | Game ROM library manager — ⬇️ down |
| **n8n** | `:5678` | Workflow automation engine — ⬇️ down |
| **FreshRSS** | `:8082` | RSS feed reader — ⬇️ down |
| **Homepage** | `:3001` | Custom service dashboard — ⬇️ down |
| **Dozzle** | `:8888` | Docker log viewer — ⬇️ down |
| **Watchtower** | — | Auto-update Docker containers |
| **PostgreSQL** | — | Database backend for Immich & other services |
| **Redis** | — | Caching layer for various services |

The Pi 5 now handles the lightweight orchestration layer — *arr stack, Zigbee, Pi-hole, RDTClient, Eufy bridge — **and took Immich back (2026-09-08)**; the remaining heavy services are still blocked on this machine.

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
