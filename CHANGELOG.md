# 📋 Changelog

> Weekly log of major changes to the Alpha Network homelab.

---

## 2026-09-20 — Two Documented Device IPs Corrected (Eufy HomeBase → `.88`, EAP 670 → `.216`); 5th Week Without DHCP Leases

### 🔧 Infrastructure
- **Eufy HomeBase 2 (T8010, MAC `90:bf:d9:34:bf:c2`) moved `10.0.1.187` → `10.0.1.88`** — identical fingerprint (dnsmasq-2.90 + RTSP `:554` + Eufy API `:9000`). Confirmed two independent ways: the ARP sweep now sees `90:bf:d9` at `.88`, and the `eufy-security-ws` bridging container logged `Connected to station T8010T15250917B0 on host 10.0.1.88 and port 21606` (2026-09-20 07:05). `.187` no longer answers ARP. Corrected in NETWORK.md (IP table + note), HARDWARE.md (topology), services/smart-home.md.
- **EAP 670 WiFi 6 AP (MAC `98:ba:5f:5b:48:0a`) moved `10.0.1.157` → `10.0.1.216`** — same MAC, still serving the TP-Link HTTPD login page on `:80`/`:443` (still the SSID `Prince Network` AP). `.157` no longer answers ARP. Corrected in NETWORK.md (IP table, note, WiFi section), services/network-services.md, docs/services-overview.md.
- **Root cause of both: ordinary DHCP churn — neither device has a static reservation** (with the lease fetch still broken we can't read OPNsense's `<staticmap>` list, but nothing else explains an unchanged MAC taking a new IP). `.88`/`.216` are today's leases; expect this to recur until the HomeBase and the AP get reservations.
- **No new machines, no service changes:** Docker on the Pi 5 still **19 containers** (unchanged since the Sep 8 Immich return); Zigbee still **8 paired devices** (Aqara temp/humidity, 2× Third Reality motion, 4× Third Reality bulbs, eWeLink contact — read from z2m `bridge/health`); HA ✅; ALpha-Server still runs Frigate only. alphapi3 (`.158`), alphamox (`.108`), both printers (`.103`/`.116`) present.
- **alphamobile-1 (Latitude, `10.0.1.176`) still offline — 31 days** now (since 2026-08-20). The "Known infra host MISSING from LAN" row fires correctly for it; its documented state (offline, Immich migrated back to the Pi 5) is accurate, so no doc change.

### 📡 Monitoring
- **No NEW infra hosts — manually confirmed (the row's "agent must confirm" case):** the OPNsense SSH lease fetch failed again, so the script fell back to the ARP sweep. All 18 undocumented live hosts were resolved by MAC/vendor/port fingerprint: 5× Amazon Fire/Echo (`08:57:fb`, `fc:49:2d`, `44:3d:54`, `a8:e6:21`, `a0:d0:dc`), 1× Apple (`98:9e:63`), Microsoft/Xbox (`1c:1a:df`), Resideo thermostat (`b8:2c:a0`), 4 randomized-MAC mobiles, 2 no-port IoT (`6c:ac:c2`, `00:33:7a`), a TP-Link Kasa plug (`78:20:51`, "SHIP 2.0" web UI), a second TP-Link plug (`d8:07:b6`, `:9999`) — **plus the two already-documented devices whose leases had moved** (EAP 670 at `.216`, HomeBase at `.88`) that this manual pass is how we caught. **All consumer/IoT: nothing to add to HARDWARE.md.**
- **OPNsense SSH lease fetch failing for the 5th consecutive week** (port 22 times out; the router's SSH daemon still appears stopped). The ARP-sweep fallback is the standing workaround; the OPNsense web UI (`https://10.0.1.1`) remains the only way to read leases, and it is why device IP churn is only caught when the sweep happens to sample the new lease.
- *arr movie/series counts still `?` (Sonarr/Radarr API keys stale — containers healthy; cosmetic, unchanged).

---

## 2026-09-13 — Immich Migrated Back to the Pi 5 (4 Containers); Docker Fleet 15 → 19

### 🔧 Infrastructure
- **Immich moved from the (offline) Latitude back to the Pi 5** — the retire-the-Latitude plan is underway. All four containers (`immich_server` v3, `immich_machine_learning`, `immich_postgres`, `immich_redis`) have been running **healthy on alphapi5 (`10.0.1.100`)** since 2026-09-08 15:16 UTC. Compose re-fetched from the official `release` template on 2026-09-08 (watchtower opt-out labels added — Immich needs manual updates because of DB migrations). Library at `/mnt/nas-data/immich/library`, DB at `/mnt/nas-data/immich/postgres`.
- **New Immich external library: the family "vault"** — `/mnt/nas-data/vault` is mounted **read-only** into `immich_server` (`/vault:ro`, added 2026-09-08) so the family archive is browsable in Immich without being mutated.
- **Docker fleet on the Pi 5: 15 → 19 containers** (20 total including Frigate on ALpha-Server). The four Immich containers are the entire delta.
- **go2rtc documented (was running undocumented since 2026-08-20):** `alexxit/go2rtc` on the Pi 5 is the RTSP restreamer behind HA/Frigate (API origin `http://10.0.1.154:8123`, the HA VM; Frigate consumes `rtsp://10.0.1.100:8554/...`). Added to the Pi 5 service inventory.
- **Still DOWN with the Latitude (offline since ~2026-08-20):** Jellyfin, RomM, n8n, FreshRSS, Homepage, Dozzle. Immich is the only service migrated so far — the rest are blocked on powering the Latitude back on or finishing the migration. NETWORK.md "Future Improvements" updated accordingly.

### 📡 Monitoring
- **No NEW infra hosts this week (confirmed manually):** the ARP/MAC sweep surfaced 16 undocumented live hosts — **all consumer-class** (Apple, Amazon Fire/Echo, Microsoft/Xbox, Resideo thermostat, TP-Link Kasa/Tapo, random-MAC mobiles/printers). No `ALpha*`/server-like OUIs — nothing that warrants a HARDWARE.md entry.
- **OPNsense SSH lease fetch still failing — 4th consecutive week** (port 22 times out; the daemon appears stopped). Web UI (`https://10.0.1.1`) remains the workaround; the weekly script keeps falling back to the ARP sweep.
- **"Known infra host MISSING from LAN" fired correctly** for `10.0.1.176` (Latitude) — expected, offline since 2026-08-20 and already documented as such (not a new outage).
- Zigbee still **8** devices, HA ✅, *arr containers UP (movie/series counts still `?` — Sonarr/Radarr API keys still stale/401).

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
