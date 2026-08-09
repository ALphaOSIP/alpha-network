# 📋 Changelog

> Weekly log of major changes to the Alpha Network homelab.

---

## 2026-08-09 — ALphaMAIN Added, VLANs Configured, Monitoring Blind Spot Fixed

### 🔧 Infrastructure
- **NEW MACHINE: ALphaMAIN** (10.0.1.141, ASUSTek OUI `30:C5:99`, MAC `30:c5:99:ef:5c:b4`) — new main machine, role TBD. First seen on DHCP Aug 9 2026 (lease renewed ~08:00 EDT), sitting on the old Omada LXC IP (.141 freed when Omada was removed).
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
