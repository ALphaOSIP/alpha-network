# Automation Services

## n8n — Workflow Automation

- **Description:** Open-source workflow automation platform.
- **Deployment:** Docker container on the Pi 5.
- **Port:** 5678 (HTTP).
- **Purpose:** Handles various automations including DeepSeek API balance monitoring.
- **Schedule:** DeepSeek balance check runs daily via cron.

### Automation Highlights

| Automation | Trigger | Description |
|---|---|---|
| DeepSeek API Balance Monitor | Daily (cron) | Checks DeepSeek API credit balance and sends alerts if low. |

---

## Cron Jobs

### Goodnight House Script

- **Host:** Mac Mini Proxmox host (alphamox)
- **Schedule:** Runs daily at 2:00 AM via root cron
- **Script Path:** `/usr/local/bin/goodnight-ha.sh`
- **Mechanism:** Calls the Home Assistant API via `curl` — independent of HA's automation engine
- **Entities controlled:**
  - `media_player.lg_webos_tv_55ur8000aua` — LG Bedroom TV
  - `light.headlight`, `light.headlight_2` — Gosund smart bulbs
  - `light.bedroom_lamp` — TP-Link KL130
- **Logging:** Output logged to `/var/log/goodnight-ha.log`
- **Purpose:** Ensures TV and lights are off overnight — energy saving and peace of mind
- **Why cron over HA automation:** More reliable — works even if HA's automation engine is reloading or has errors. Runs directly against the HA REST API.

### Network Monitor Script

- **Host:** Pi 5.
- **Function:** Periodically checks critical services on the LAN.
- **Monitored Services:**
  - Jellyfin
  - RomM
  - Pi-hole
  - (Other critical services)
- **Alerting:** Sends push notifications via **Ntfy** if any monitored service is unreachable or down.

---

## Related

- See [storage.md](./storage.md) for RomM (monitored by this script).
- See [networking.md](../networking.md) for Ntfy alert configuration.
