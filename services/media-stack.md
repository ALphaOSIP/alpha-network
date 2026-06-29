# Media Automation Stack

> Raspberry Pi 5 · Docker-based · Real-Debrid architecture

## Overview

The media automation stack runs entirely in Docker on a Raspberry Pi 5 (ARM64). It manages the lifecycle of TV shows and movies — from automated searching and downloading to organization, streaming, and subtitle management. All services are interconnected via the `arr_net` Docker bridge network.

The key architectural distinction of this setup is the use of **Real-Debrid** as the download backend instead of a traditional VPN-bound torrent client. This eliminates the need for a VPN, port forwarding, and the bandwidth overhead of P2P traffic while providing near-instantaneous downloads via cached content on Real-Debrid's servers.

**Pipeline Note (June 2026):** RDTClient uses a **watch folder** approach instead of direct qBittorrent API integration. Sonarr/Radarr write `.torrent` files to a watched directory via the Torrent Blackhole download client, RDTClient picks them up, sends them to Real-Debrid, and pulls completed files back. See the [Data Flow](#data-flow) section for details.

---

## Architecture Diagram

```
                         ┌──────────────────┐
                         │   Prowlarr (9696) │
                         │   Indexer Mgmt    │
                         └──────┬──┬────────┘
                                │  │  (indexer feeds)
              ┌─────────────────┘  └─────────────────┐
              ▼                                       ▼
   ┌──────────────────┐                   ┌──────────────────┐
   │   Sonarr (8989)   │                   │   Radarr (7878)   │
   │   TV Shows        │                   │   Movies          │
   └──────┬───────────┘                   └──────┬───────────┘
          │  (subtitle requests)                  │  (subtitle requests)
          └───────────┬───────────┘
                      ▼
           ┌──────────────────┐
           │   Bazarr (6767)   │
           │   Subtitles       │
           └──────────────────┘

   ┌──────────────────┐
   │ FlareSolverr(8191)│◄──── Bypass Cloudflare for indexers
   └──────────────────┘

          │  (release grabs)
          ▼
   ┌──────────────────┐
   │ RDTClient (6500)  │
   │ Real-Debrid Agent │
   └──────┬───────────┘
          │  (instant download via RD CDN)
          ▼
   ┌──────────────────┐
   │   Jellyfin (8096) │
   │   Media Server    │
   └──────────────────┘
          │
          ▼
   ┌──────────────────┐
   │ LG TV · Phones    │
   │ Browsers (direct) │
   └──────────────────┘

         arr_net Docker Bridge Network
   ─────────────────────────────────────────
```

---

## Services

### Jellyfin — Media Streaming Server

- **Port:** `8096`
- **Role:** Central media server. Streams movies and TV shows to clients via direct play or transcoding.
- **Clients:** LG TV (webOS app), mobile phones (Jellyfin app), web browsers.
- **Hardware Acceleration:** Not configured. The Pi 5 has a VideoCore VII GPU capable of hardware-accelerated transcoding, but this is not currently set up. All direct-play content works fine; software transcoding of unsupported codecs may be CPU-intensive on the Pi 5.
- **Library Paths:** Points to the media directories populated by Sonarr/Radarr via RDTClient.
- **Docker Image:** `linuxserver/jellyfin:latest` (ARM64)

#### Configuration Notes

- No hardware acceleration enabled — consider enabling `jellyfin-ffmpeg6` with VAAPI or V4L2 if transcoding is needed.
- Direct play to LG TV works well with H.264/AAC content in MKV or MP4 containers.
- Remote access via reverse proxy (not configured yet) or tailscale.

---

### Sonarr — TV Show Automation

- **Port:** `8989`
- **Role:** Monitors TV show library, searches for wanted episodes, sends grabs to download client, and organizes completed downloads into the media library.
- **Configuration:**
  - Download client: **Torrent Blackhole** — writes `.torrent` files to `/downloads/watch/`
  - Category: `tv-sonarr`
  - Indexers: Provided by Prowlarr (sync enabled)
  - Root folder: `/tv`
  - Quality profile: HD - 720p/1080p
- **Current library:** Bleach (seasons 1-17, only S17 TYBW monitored)
- **Docker Image:** `linuxserver/sonarr:latest` (ARM64)

---

### Radarr — Movie Automation

- **Port:** `7878`
- **Role:** Same concept as Sonarr but for movies. Monitors wanted films, searches indexers, sends to download client.
- **Configuration:**
  - Download client: **Torrent Blackhole** — writes `.torrent` files to `/downloads/watch/`
  - Category: `radarr`
  - Indexers: Provided by Prowlarr (sync enabled)
  - Root folder: `/movies`
  - Quality profile: HD - 720p/1080p
- **Current library:** Godzilla Minus One (2023)
- **Docker Image:** `linuxserver/radarr:latest` (ARM64)

---

### Bazarr — Subtitle Management

- **Port:** `6767`
- **Role:** Automatically searches for and downloads subtitles for content managed by Sonarr and Radarr.
- **Configuration:**
  - Connected to Sonarr and Radarr APIs for library awareness
  - Languages: English (primary), with support for additional languages as needed
  - Providers: OpenSubtitles, Podnapisi, Subscene, YIFY Subtitles, etc.
  - Automatic download on missing subtitles, periodic re-scan of entire library
- **Docker Image:** `linuxserver/bazarr:latest` (ARM64)

---

### Prowlarr — Indexer Manager

- **Port:** `9696`
- **Role:** Central indexer management. Connects to torrent/usenet indexers and syncs them to Sonarr, Radarr, and other *arr apps.
- **Configuration:**
  - Indexers: Public trackers, private indexers, Usenet indexers (as available)
  - FlareSolvr integration: Routes indexer requests through FlareSolverr when indexers are behind Cloudflare protection
  - Sync apps: Sonarr, Radarr (automatic indexer sync)
- **Docker Image:** `linuxserver/prowlarr:latest` (ARM64)

---

### RDTClient — Real-Debrid Download Client

- **Port:** `6500`
- **Role:** Download client that integrates with Real-Debrid's API. Instead of traditional torrenting, it sends magnet links or file hosts to Real-Debrid, which returns instantly downloadable content (cached on their servers).
- **Why Real-Debrid Instead of a VPN:**
  - **No VPN required:** Real-Debrid handles all P2P traffic on their infrastructure; your IP is never exposed to swarm participants.
  - **Instant downloads:** If a torrent is cached on Real-Debrid (most popular content is), download starts immediately at full line speed via HTTPS.
  - **No port forwarding:** Traditional torrent clients need open ports; Real-Debrid avoids this entirely.
  - **Lower bandwidth usage:** Direct HTTPS download instead of P2P upload overhead.
  - **Cost:** ~$3/month vs a VPN ($5-10/month) + seedbox (optional).
  - **Limitations:** Relies on Real-Debrid's cached content availability; less common/niche content may not be cached and requires queuing on their servers.
- **Configuration:**
  - API Token: Configured with active Real-Debrid account
  - **AutoImport: Enabled** (critical — was the root cause of the pipeline being broken)
  - **Watch Folder:** `/downloads/watch/`
  - **Download Path:** `/downloads/`
  - **Categories:** `tv-sonarr` → `/downloads/tv-sonarr/`, `radarr` → `/downloads/radarr/`
  - **Download Limit:** 8 concurrent
  - **Sonarr/Radarr integration:** Via Torrent Blackhole (not qBittorrent API — auth mismatch issues)
- **Docker Image:** `rogerfar/rdtclient:latest` (ARM64)

---

### FlareSolverr — Cloudflare Bypass Proxy

- **Port:** `8191`
- **Role:** Proxy server that solves Cloudflare challenges (JS challenges, CAPTCHAs) so the *arr apps can access indexers protected by Cloudflare's anti-bot measures.
- **Configuration:**
  - Default mode: Proxy all requests through FlareSolverr
  - Timeout: 60 seconds (configurable)
  - Prowlarr integration: Configured as a HTTP proxy in Prowlarr's indexer settings
- **Docker Image:** `flaresolverr/flaresolverr:latest` (ARM64)

---

## Docker Networking

All services run on the **`arr_net`** Docker bridge network, enabling internal DNS resolution by container name (e.g., Sonarr talks to `rdtclient:6500`, Prowlarr talks to `flaresolverr:8191`).

**Network configuration:**
```yaml
networks:
  arr_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

**Port exposure** is limited to host-bound ports for reverse proxy/admin access (not exposed to WAN directly without a proxy).

---

## Data Flow

1. **User adds a show/movie** in Sonarr/Radarr
2. **Sonarr/Radarr** queries Prowlarr for indexer results
3. **Prowlarr** proxies requests through FlareSolverr if needed
4. **Sonarr/Radarr** selects the best release and writes a `.torrent` file to `/downloads/watch/`
5. **RDTClient** detects the file in the watch folder and sends the magnet to Real-Debrid
6. **Real-Debrid** processes the torrent on their servers (cached content is instant)
7. **RDTClient** (AutoImport) downloads the completed file via HTTPS to the category folder
8. **Sonarr/Radarr** scans the download folder and imports the file into the organized library
9. **Bazarr** detects the new file and fetches subtitles
10. **Jellyfin** notices the library change and updates its catalog
11. **User streams** via Jellyfin on LG TV, phone, or browser

---

## Current Status

| Service      | Status | Notes                                    |
|--------------|--------|------------------------------------------|
| Jellyfin     | ✅     | Direct play only; no HW transcoding yet  |
| Sonarr       | ✅     | Torrent Blackhole → RDTClient watch folder |
| Radarr       | ✅     | Torrent Blackhole → RDTClient watch folder |
| Bazarr       | ✅     | Connected to Sonarr/Radarr APIs          |
| Prowlarr     | ✅     | 8 indexers synced to Sonarr + Radarr     |
| RDTClient    | ✅     | AutoImport=on, watch folder active       |
| FlareSolverr | ✅     | Running as proxy for Prowlarr            |

### Current Content

| Title | Type | Added | Status |
|-------|------|-------|--------|
| Godzilla Minus One (2023) | Movie | June 26, 2026 | Searching |
| Bleach — TYBW S17 (Cour 1) | TV | June 26, 2026 | Monitoring |

---

## Future Improvements

- [ ] Enable Jellyfin hardware acceleration (VAAPI/V4L2 on Pi 5)
- [ ] Add reverse proxy (Nginx Proxy Manager / Caddy / Traefik) for secure remote access
- [ ] Set up Tailscale/WireGuard for private remote streaming
- [ ] Add monitoring (healthchecks, disk usage alerts)
- [ ] Configure automatic backups of *arr databases and configs
- [ ] Evaluate transitioning to Jellyseerr/Overseerr for user-facing request management
- [ ] Add Tdarr for automated media transcoding/optimization
