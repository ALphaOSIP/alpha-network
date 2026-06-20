# Storage Services

## Immich — Photo Management

- **Description:** Open-source Google Photos alternative for self-hosted photo and video management.
- **URL:** [http://10.0.1.100:2283](http://10.0.1.100:2283)
- **Backend Components:**
  - **PostgreSQL** — Primary database for metadata and application state.
  - **Redis** — Caching and job queue for background tasks.
- **Storage:** Stores uploaded photos and videos on local/mounted storage.
- **Machine Learning Capabilities:**
  - Facial recognition (automatic person tagging).
  - Object detection and scene classification.
  - Image similarity search.
- **Purpose:** Centralized photo library with search, albums, sharing, and mobile app access.

---

## RomM — Game ROM Manager

- **Description:** Open-source ROM (Read-Only Memory) manager for organizing video game collections.
- **URL:** [http://10.0.1.100:3000](http://10.0.1.100:3000)
- **Database:** MariaDB.
- **ROM Storage Path:** `/mnt/nas-data/media/roms/`

### Collection Stats

| Metric | Value |
|---|---|
| Total Games | 36+ |
| Platforms | 6 |
| Largest Platform | Switch (~55 GB) |

### Supported Platforms

| Platform | Notes |
|---|---|
| Nintendo Switch | ~55 GB of ROMs |
| Nintendo DS (NDS) | |
| PlayStation 1 (PS1) | |
| Game Boy Advance (GBA) | |
| Super Nintendo (SNES) | |
| Mods | Custom game modifications |

---

## Samba File Sharing

- **Description:** SMB/CIFS network share for file access across the LAN.
- **Share Path:** `\\10.0.1.100\shared`
- **Firewall (UFW):** Ports **137–139** (NetBIOS) and **445** (SMB) are open.
- **Access:** Available to all devices on the local LAN.
- **Purpose:** General-purpose file sharing between Windows, macOS, and Linux clients on the network.

---

## Related

- See [automation.md](./automation.md) for the network monitor that checks RomM availability.
- See [networking.md](../networking.md) for firewall rules and share access restrictions.
