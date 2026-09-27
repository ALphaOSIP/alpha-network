# Dell Latitude 5501 — Heavy Lifter

> **✅ ONLINE — network link restored 2026-09-24 at `10.0.1.134`** (was `.176`). The Aug 20–Sep 24 "outage" was a **link-layer failure, not a power-off** — `eno2` dropped 2026-08-20 ~10:56 UTC and the laptop ran headless/unreachable for ~5 weeks; it never rebooted (uptime ~6.5 weeks, boot 2026-08-12). On 2026-09-24 the link returned, it took a new DHCP lease (`.176` → `.134`) and Tailscale/SSH reconnected. **All services are up again** (Jellyfin, RomM, n8n, FreshRSS, Homepage, Dozzle, Immich stack, Watchtower — 11 containers). Note: the Pi 5 gained its own Immich stack on 2026-09-08 while this box was unreachable, so **two Immich instances now exist**. See [CHANGELOG.md](../CHANGELOG.md).

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
| **Network** | Gigabit Ethernet (eno2) — `10.0.1.134` (DHCP; was `.176` until 2026-09-24) |
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

The Latitude hosts the heavyweight services that *need* x86 power — **all running again since the link came back on 2026-09-24 (11 containers, up continuously since the 2026-08-12 boot)**:

| Service | Port | Purpose |
|---|---|---|
| **Jellyfin** | `:8096` | Media streaming & hardware-accelerated transcoding — ✅ up |
| **Immich** | `:2283` | Photo ML (facial recognition, object detection) + `immich_postgres` + `immich_redis` — ✅ up ⚠️ *a second, separate Immich stack also runs on the Pi 5 since 2026-09-08 — reconcile* |
| **RomM** | `:3000` | Game ROM library manager — ✅ up (+ `romm-db`) |
| **n8n** | `:5678` | Workflow automation engine — ✅ up |
| **FreshRSS** | `:8082` | RSS feed reader — ✅ up |
| **Homepage** | `:3001` | Custom service dashboard — ✅ up |
| **Dozzle** | `:8888` | Docker log viewer — ✅ up |
| **Watchtower** | — | Auto-update Docker containers — ✅ up |
| **PostgreSQL** | — | Database backend for Immich & other services — ✅ up |
| **Redis** | — | Caching layer for various services — ✅ up |

The Pi 5 handles the lightweight orchestration layer — *arr stack, Zigbee, Pi-hole, RDTClient, Eufy bridge — **plus a second Immich stack it picked up on 2026-09-08** while the Latitude was unreachable. The Latitude's heavy services are all live again at `.134`.

> **⚠️ Monitoring gap (2026-09-27):** Uptime Kuma's eight Latitude monitors (Jellyfin, RomM, Immich, FreshRSS, n8n, Homepage, Dozzle, Watchtower) plus the `alphamobile (Latitude)` ping still point at the **dead `10.0.1.176`** — they all read red even though the services are up. Repoint them to `10.0.1.134` (and ideally add DHCP reservations so this stops recurring).

---

## Network Note

Due to a switch-level quirk, the Pi 5 (`10.0.1.100`) historically could not directly reach the Latitude on the local subnet. All cross-host communication routes through Tailscale instead — the Latitude is accessed at `100.82.167.20` from the Pi 5 side. This doesn't affect normal operation (all Docker services bind to their ports), but any host-to-host scripts or monitoring use the Tailscale IP. (After the 2026-09-24 re-lease the Pi's ARP table does now see `10.0.1.134` on `eth0`, but Tailscale remains the documented path.)

---

## Quick Reference

| Command | Purpose |
|---|---|
| `ssh alpha@100.82.167.20` | SSH via Tailscale (preferred; key `id_hermes`) |
| `ssh alpha@10.0.1.134` | SSH over LAN (current lease; was `.176` before 2026-09-24) |
| `docker compose up -d` | Start/update a service |
| `tailscale ping 100.82.167.20` | Test connectivity from Pi 5 |
