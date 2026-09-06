# 📋 Changelog

> Weekly log of major changes to the Alpha Network homelab.

---

## 2026-09-06 — Latitude (alphamobile-1) OFFLINE Since Aug 20, Heavy Services Down; Monitoring Blind Spot #3 Fixed

### 🔧 Infrastructure
- **alphamobile-1 (Dell Latitude 5501, `10.0.1.176` / Tailscale `100.82.167.20`) has been OFFLINE since ~2026-08-20 03:25 EDT** — 17 days as of this check. Not on the LAN at all today (no ARP reply, absent from the root ARP sweep, direct TCP to its ports fails). Uptime Kuma flipped to `EHOSTUNREACH` at 2026-08-20 03:25 after being green Aug 5–19; Tailscale control plane agrees (`offline, last seen 17d ago`).
- **All Latitude-hosted services are DOWN with it:** Jellyfin, Immich (photos/ML + its Postgres/Redis), n8n, FreshRSS, RomM, Homepage, Dozzle, Watchtower. Verified none of them moved: ALpha-Server (`.135`) runs **only Frigate** (its `services/` dir has immich/jellyfin/n8n compose configs — NOT running), and the Pi 5 doesn't serve them either.
- **Timing suggests a relation to the ALpha-Server standup (~Aug 20):** likely powered down when the new box came up, but the migration is unfinished (Frigate only). **Action needed:** power the Latitude back on, or migrate Jellyfin/Immich/n8n/FreshRSS/RomM to ALpha-Server and retire it (added as a NETWORK.md "Future Improvements" item).
- Docs corrected from "✅ Online" to **🔴 OFFLINE since 2026-08-20** across HARDWARE.md (summary, section, topology, Power & Environment), NETWORK.md (topology, role/IP tables, Tailscale table), README.md, and hardware/latitude.md. NETWORK.md's stale "Key Services (alphapi5)" table (claimed Jellyfin/RomM/Immich run on the Pi) fixed to the real Pi fleet (*arr, Zigbee/MQTT, RDTClient, Portainer, Uptime Kuma).
- **No NEW infra hosts this week:** full ARP/MAC sweep (~27 live hosts) — every host outside the documented inventory is consumer-class (5× Amazon Fire TV/Echo, 2× TP-Link Kasa/Tapo, Apple TV/phone-class, Resideo thermostat, random-MAC mobiles, printers, Eufy HomeBase). ALphaMAIN (`.233`) not on LAN — normal (usually off). `.252` answers ICMP but has no MAC/ARP entry — router alias/phantom, not a host.

### 📡 Monitoring
- **Blind spot #3 found & fixed — "known host vanished" was invisible:** the weekly check only ever flagged *new* IPs, so a documented host dropping off the network (Latitude, ~Aug 20) sailed through the Aug 23 push and the Aug 30 run undetected while docs claimed ✅ Online. `weekly-doc-check.py` now runs a root ARP sweep of `10.0.1.0/24` every week and reports **"Known infra host MISSING from LAN"** for any always-on host (OPNsense, alphapi5, alphamox, ALpha-Server, HA VM, Latitude) not answering ARP. Re-run today correctly flags `10.0.1.176`.
- **OPNsense SSH lease fetch still failing — 3rd consecutive week** (port 22 times out from alphapi5 *and* ALpha-Server; the anti-lockout rule allows 22 on LAN, so the router's SSH daemon is likely just stopped). The Aug 23 fix only made the failure visible — it never restored data. Script now falls back to the ARP sweep on fetch failure and lists every undocumented live host with MAC + vendor (this week: 14, all consumer-class). NETWORK.md maintenance notes updated: use the OPNsense web UI (`https://10.0.1.1`).
- *arr API keys stale (cosmetic): the status script and Uptime Kuma both get 401 from Sonarr/Radarr (the containers themselves are UP) — movie/series counts show `?` until keys are refreshed. Zigbee still 8, Docker still 15, HA ✅.*

---

## 2026-08-23 — ALpha-Server Discovered & Documented, ALphaMAIN IP Moved, Frigate NVR Live

### 🔧 Infrastructure
- **NEW MACHINE: ALpha-Server** (`10.0.1.135`, MAC `d4:5d:64:aa:45:b1` ASUSTek, AMD Ryzen 5 3600 6C/12T, 16 GB RAM, ~256 GB SSD, AMD RX 580) — **Ubuntu 24.04 x86_64 Frigate NVR box**. Set up ~Aug 1-4 2026 (home dir + `alpha-server` SSH alias on alphapi5 date to Aug 2) but **was never documented**: the Aug 9 LAN investigation knew the IP (it's in the weekly check's `KNOWN_HOSTS`) yet it never reached HARDWARE.md/NETWORK.md. Found 2026-08-23 via ARP/MAC scan after the weekly lease fetch failed.
- **New service: Frigate NVR** on ALpha-Server (`ghcr.io/blakeblackshear/frigate:stable`, healthy) — AI object detection on the Eufy camera RTSP streams (`rtsp://10.0.1.100:8554/...`), VAAPI hw accel on the RX 580, 14-day alert retention. First new service host since the Latitude. Tailscale `100.123.100.38`. Hermes + Claude Code installed; `lan-scan/` tooling lives here; `services/` dir also has (not-yet-running) configs for baby-tracker/immich/jellyfin/n8n.
- **ALphaMAIN DHCP IP changed: `10.0.1.141` → `10.0.1.233`** (verified via ARP 2026-08-23 — `30:c5:99:ef:5c:b4` now on `.233`, `.141` empty). `.141` is now free; all Omada-era `.141:8088` references cleaned (network-services.md, mac-mini.md, services-overview.md, topology.svg → ALpha-Server box).
- HARDWARE.md/NETWORK.md/README.md updated: ALpha-Server added (summary, topology, IP tables, Tailscale table); ALphaMAIN IPs corrected everywhere; smart-home.md gains a Frigate section.

### 📡 Monitoring
- **Weekly check blind spot #2 fixed:** this week's report said "NEW infra hosts: none" even though ALpha-Server was on the LAN and ALphaMAIN had moved — the OPNsense lease fetch failed and the script only logged the failure under the consumer-devices row, so the infra row printed a false `none`. The script now reports `UNKNOWN (lease fetch failed)` in the infra row on fetch failure, prompting manual investigation (that's how ALpha-Server was found this week).
- Weekly check `KNOWN_HOSTS` updated: `.141` → `.233` (ALphaMAIN), `.135` retained (now documented).
- Daily connectivity check updated: ALphaMAIN ping target `.141` → `.233`; ALpha-Server added via SSH (`alphaserver@10.0.1.135`).
- Also fixed a stale doc claim: alphapi3 is online at `.158` (HARDWARE.md "Power & Environment" said powered off).

---

## 2026-08-09 — ALphaMAIN Added, VLANs Configured, Monitoring Blind Spot Fixed

### 🔧 Infrastructure
- **NEW MACHINE: ALphaMAIN** (10.0.1.141, ASUSTek OUI `30:C5:99`, MAC `30:c5:99:ef:5c:b4`) — **Joseph's personal gaming PC (Windows)**. First seen on DHCP Aug 9 2026, sitting on the old Omada LXC IP (.141 freed when Omada was removed). Policy: NO agents/services/Hermes installs inside — SSH key access (`id_alphamain` on alphapi5) for connectivity checks + on-request help only.
- **Monitoring blind spot found & documented:** weekly doc cron reported "no new hardware" because the check only counts Zigbee/*arr/Docker — it never scans the LAN. ALphaMAIN was invisible to all monitoring. Fix: LAN host scan added to weekly cron + ALphaMAIN added to daily connectivity check.
- **VLANs now live on OPNsense:** vlan01 Trusted (10.0.10.0/24), vlan02 Services (10.0.20.0/24), vlan03 IoT (10.0.30.0/24), vlan04 Guest (10.0.40.0/24). All four gateways UP. No devices moved yet.
- Pi 3 confirmed back online at **10.0.1.158** (docs said .101/offline)
- EAP 670 confirmed at **10.0.1.157** (docs said DHCP-assigned unknown)
- Omada references cleaned from docs (controller removed everywhere Aug 2026)

### 🏠 Smart Home
- Zigbee verified at **8 paired devices** (1 Aqara WSDCGQ11LM, 2 Third Reality 3RMS16BZ motion, 4 Third Reality 3RCB01057Z bulbs, 1 eWeLink CK-TLSR8656)

### 📡 Network Audit (2026-08-09)
Full DHCP lease table reviewed: 30+ devices on LAN including phones (S21 Ultra, Z-Fold3, S23 Ultra, iPhone, Watch), Fire TV Cube 2022, Fire TV Stick 4K, Echo/Fire TV devices, Kasa smart plug (HS300), Honeywell thermostat (Resideo), Xbox, LG TV (LG Innotek), tablets. Consumer devices — not tracked in repo inventory.

---

## 2026-07-19 — 8th Zigbee Device, Docker Fleet Grows to 15

### 🏠 Smart Home
- **1 new Zigbee device paired** (now 8 total) — device type unknown
- **Home Assistant** continues running stable
- Tuya/Kasa integrations still pending re-add from June rollback

### 🎬 Media Stack
- **Sonarr** still tracking 3 series — no change
- **Radarr** count could not be fetched this week

### 🔧 Infrastructure
- **15 Docker containers** running (was 14) — 1 new service added
- All services healthy

---

## 2026-07-05 — More Zigbee Devices, Media Library Growth

### 🏠 Smart Home
- **3 new Zigbee devices paired** (now 7 total):
  - 2× TRÅDFRI color bulbs (#3 and #4) — hue/sat/brightness
  - 1× Third Reality motion sensor (#2)
- **Home Assistant** continues running fine, no rollback issues this week
- Tuya/Kasa integrations still pending re-add from June rollback

### 🎬 Media Stack
- **Radarr expanded to 5 movies** (was 1):
  - **Michael** (2026) — Michael Jackson biopic, monitored
  - **Hoppers** (2026) — Pixar film, monitored
  - **The Drama** (2026) — new release, monitored
  - **Pulp Fiction** (1994) — added to library (unmonitored)
  - Godzilla Minus One (2023) — still monitoring
- **Sonarr expanded to 3 series** (was 1):
  - **Euphoria (US)** — HBO — S3 monitored, 6/8 eps downloaded (13.2 GB)
  - **Industry** (2020) — HBO — S4 monitored
  - Bleach (2004) — S17 TYBW monitored (existing)
- **All *arr services** healthy and downloading

### 🔧 Infrastructure
- **14 Docker containers** running stable (was previously listed as 25+ in README — corrected)
- All services healthy across the stack

---

## 2026-06-28 — Zigbee Network Live & *Arr Pipeline Fix

### 🏠 Smart Home
- **MQTT connected** to HA via UI (10.0.1.100:1883, no auth) — wired up successfully
- **4 Zigbee devices paired:**
  - Aqara temp/humidity sensor
  - Third Reality motion sensor
  - 2× TRÅDFRI color bulbs (hue/sat/brightness)
- **Permit join** enabled/disabled to add bulbs
- **Tuya/Kasa integrations** lost in HA snapshot rollback (need re-add)
- **Snapshot incident:** HA VM rolled back to `pre-ha-setup` after config.yaml MQTT modification caused boot loop. Lesson: always configure MQTT via HA UI in 2026.x.

### 🎬 Media Stack
- **RDTClient pipeline fixed:**
  - Root cause: `AutoImport=False` — RDTClient never pulled files from Real-Debrid
  - Switched to **Torrent Blackhole** watch folder approach (more reliable than qBittorrent API)
  - Categories mapped: `tv-sonarr` → `/downloads/tv-sonarr/`, `radarr` → `/downloads/radarr/`
  - Download limit bumped to 8
- **Godzilla Minus One** added to Radarr (HD-1080p, searching)
- **Bleach TYBW S17** added to Sonarr (monitoring, old seasons unmonitored)
- **8 indexers** synced via Prowlarr to both Sonarr and Radarr

### 🔧 Infrastructure
- **GitHub SSH key** added for agent push access
- **Weekly documentation cron** configured (Sundays)

---

## 2026-06-21 — Tailscale Routing Fix & Alexa Setup

- Tailscale `--accept-routes=false` documented after routing fix
- Alexa Media Player v5.15.4 custom components installed in HA
- Goodnight House automation documented with specific entities
- WireGuard tunnel status confirmed working but superseded by Tailscale

---

## Earlier

See the repo's commit history for earlier changes. Going forward, significant updates are logged here weekly.
