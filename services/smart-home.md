# Smart Home Setup

## Overview

Home Assistant OS running as a Proxmox VM (ID 100) on a Mac Mini Proxmox host (`alphamox`). Accessible at [http://10.0.1.154:8123](http://10.0.1.154:8123).

- **HA OS Version:** 2026.6.4
- **VM Specs:** 4 GB RAM allocated
- **No Nabu Casa subscription** — remote access via Tailscale subnet router on the Mac Mini

---

## Integrations

### Zigbee — Local IoT Mesh

A Sonoff Zigbee 3.0 USB Dongle Plus V2 (ZBDongle-E, EmberZNet 7.4.5 firmware) is connected to the Raspberry Pi 5 at `/dev/ttyUSB0`.

- **Coordinator:** Zigbee2MQTT (Docker on Pi 5, port 8080)
- **MQTT Broker:** Mosquitto (Docker on Pi 5, port 1883) — `allow_anonymous: true`
- **HA Connection:** MQTT integration added via HA UI (broker `10.0.1.100:1883`, no auth)
- **Network:** PAN 34179, channel 11
- **Status:** HA auto-discovers devices via MQTT homeassistant discovery topic

#### Paired Devices

| Device | Address | Type | Status |
|--------|---------|------|--------|
| **Aqara WSDCGQ11LM** | `0x00158d008c7eae5d` | Temp/humidity/pressure sensor | ✅ Paired |
| **Third Reality 3RMS16BZ** | `0xb40e060fffe707f1` | Wireless motion sensor | ✅ Paired |
| **TRÅDFRI Bulb 1** | `0x7cb94c6867d60000` | Color bulb (hue/sat/brightness) | ✅ Paired |
| **TRÅDFRI Bulb 2** | `0x7cb94c6803180000` | Color bulb (hue/sat/brightness) | ✅ Paired |

**Pending:** 3 motion sensors, 1 humidity sensor, 2 door sensors — waiting to be paired.

#### Pairing New Devices

To add a new Zigbee device:
1. Enable permit_join in zigbee2mqtt (or ask the agent)
2. Put the device in pairing mode (usually 6x on/off for bulbs, button press for sensors)
3. Device appears within 30 seconds
4. Disable permit_join afterward

### MQTT Broker

- **Host:** `10.0.1.100:1883` (Mosquitto on Pi 5)
- **WebSocket:** `10.0.1.100:9001`
- **Auth:** None (internal network only)
- **Configured via:** HA UI Settings → Devices & Services → Add Integration → MQTT

### Eufy Security

- **Cameras (5 total):**
  - Driveway
  - Front Door
  - Side Door
  - Backyard
  - Doorbell
- **HomeBase:** Model T8010 at `10.0.1.187`
- **Connection:** Cloud bridge via `eufy-security-ws` Docker container on the Pi 5 (port 3002). Handles 2FA and trusted device registration.

### Tuya IoT Cloud *(needs re-add)*

- Lost during HA snapshot rollback on June 25
- **Previously:** 2 Gosund smart bulbs (`light.headlight`, `light.headlight_2`)
- **To restore:** Re-add Tuya Smart integration via HA UI (cloud-based, same credentials)

### TP-Link Kasa *(needs re-add)*

- **KL130 smart bulb** (`light.bedroom_lamp`) — lost during rollback
- **To restore:** Re-add Kasa integration via HA UI

### LG webOS Smart TV

- **Model:** 55UR8000AUA
- **Entity:** `media_player.lg_webos_tv_55ur8000aua`
- **IP:** `10.0.1.143`
- **Status:** Retained through rollback (integration may need re-auth)

### HACS (Home Assistant Community Store)

- Custom component repository installed.
- Survived rollback (lives in `custom_components/`).

### Alexa Media Player

- **Version:** v5.15.4
- **Status:** Custom component files installed in HA `custom_components/`
- **Devices:** Amazon Fire TV Cube (2nd Gen) in bedroom, plus other Echo devices
- **Setup status:** Pending — needs one-time Amazon OAuth login through HA UI
- Survived rollback (files in `custom_components/`)

---

## Automations

### Goodnight House

- **Trigger:** Daily at 2:00 AM (cron on Mac Mini host)
- **Actions:** Turns off all lights and the TV via HA REST API
- **Status:** Functioning — targets need updating if light entities change after re-adding integrations
- **Why cron over HA:** More reliable — works even if HA's automation engine is reloading

---

## Dashboards

- **Custom Home Dashboard** has sections for Security (camera feeds), Lights, and TV.
- **Camera Tab:** Dedicated tab with a 2-column grid layout for live camera views.
- Both were lost in the rollback and need rebuilding if desired.

---

## Remote Access

- **Method:** Tailscale subnet router running on the Mac Mini Proxmox host.
- **No Nabu Casa subscription** — all external access is through the Tailscale tunnel.

---

## Infrastructure Notes

- **Zigbee coordinator** runs on Pi 5, not in HA VM — this avoids USB passthrough complications and keeps the network stable across HA reboots/updates.
- **MQTT bridge** connects HA to Zigbee2MQTT — configured purely through the UI (config.yaml modification broke HA 2026.x and caused a boot loop).
- **Snapshot rollback:** On June 25, the HA VM was restored to the `pre-ha-setup` snapshot after config changes caused a boot loop. This wiped all UI-added integrations (Tuya, Kasa, dashboards) but preserved custom_components (Alexa Media, Eufy, HACS).
