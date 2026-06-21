# Smart Home Setup

## Overview

Home Assistant OS v2026.4.3 running as a Proxmox VM (ID 100) on a Mac Mini Proxmox host. Accessible at [http://10.0.1.154:8123](http://10.0.1.154:8123).

No Nabu Casa subscription. Remote access is handled via a **Tailscale subnet router** running on the Mac Mini.

---

## Integrations

### Eufy Security

- **Cameras (5 total):**
  - Driveway — 100% battery
  - Front Door — 74% battery
  - Side Door — 100% battery
  - Backyard — 33% battery
  - Doorbell — 76% battery
- **HomeBase:** Model T8010 at `10.0.1.187`
- **Connection:** Cloud bridge via `eufy-security-ws` Docker container running on the Raspberry Pi 5 (port 3002). Handles 2FA and trusted device registration.

### Tuya IoT Cloud

- **2 Gosund smart bulbs**
  - `light.headlight`
  - `light.headlight_2`

### TP-Link Kasa

- **KL130 smart bulb**
  - `light.bedroom_lamp`

### LG webOS Smart TV

- **Model:** 55UR8000AUA
- **Entity:** `media_player.lg_webos_tv_55ur8000aua`
- **IP:** `10.0.1.143`

### HACS (Home Assistant Community Store)

- Custom component repository installed.

### Alexa Media Player

- **Version:** v5.15.4 (latest release, May 2026)
- **Status:** Custom component files installed in HA `custom_components/`
- **Devices:** Amazon Fire TV Cube (2nd Gen) in bedroom, plus other Echo devices
- **Features planned:** TTS announcements, daily briefings, voice control of HA entities via Alexa routines
- **Setup status:** Pending — needs to be configured through HA Settings → Devices & Services → Add Integration → "Alexa Media Player" with Amazon account credentials. Requires one-time Amazon OAuth login through the HA UI.

---

## Automations

### Goodnight House

- **Trigger:** Daily at 2:00 AM
- **Actions:** Turns off all lights and the TV.
- **Purpose:** Ensures nothing is left running overnight.

---

## Dashboards

### Custom Home Dashboard

- **Sections:**
  - Security (camera feeds)
  - Lights
  - TV
- **Camera Tab:** Dedicated tab with a 2-column grid layout for live camera views.

---

## Remote Access

- **Method:** Tailscale subnet router running on the Mac Mini Proxmox host.
- **No Nabu Casa subscription** — all external access is through the Tailscale tunnel.
