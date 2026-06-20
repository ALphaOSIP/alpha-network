# Raspberry Pi 3 Model B+ Rev 1.3

## Overview

A Raspberry Pi 3 Model B+ (Revision 1.3) that was previously deployed as a K3s worker node in the home Kubernetes cluster. It is currently **powered off / offline** and is slated for a new project: a portable LibreELEC + Jellyfin media stick for hotel use.

## Hardware

| Attribute | Value |
|-----------|-------|
| **Model** | Raspberry Pi 3 Model B+ Rev 1.3 |
| **RAM** | 1 GB |
| **Storage** | 117 GB SD card (3.4 GB used) |
| **MAC Address** | `B8:27:EB:CF:91:C8` |
| **Last Seen Temp** | 48°C (idle) |

## Software

| Attribute | Value |
|-----------|-------|
| **OS** | Ubuntu 24.04 |
| **IP Address** | 10.0.1.101 (static) |
| **Status** | **OFFLINE** / powered off |

## Previous Role

- Deployed as a **K3s worker node** in the home Kubernetes cluster.
- At the time of decommissioning, no pods were scheduled on this node.

## Planned Project

The Pi 3 will be repurposed as a **travel media stick**:

- **OS**: [LibreELEC](https://libreelec.tv/) — a lightweight Kodi-centric Linux distribution
- **Media Server**: [Jellyfin](https://jellyfin.org/) for streaming local media
- **Use Case**: Hotel room entertainment — plug in via HDMI, connect to hotel Wi-Fi, stream movies/shows from the SD card
- **Remote Access**: [Tailscale](https://tailscale.com/) for SSH access while on the road

## Notes

- The 117 GB SD card provides ample space for a modest media library alongside the OS.
- With only 3.4 GB currently used on Ubuntu, the LibreELEC swap should free up significant capacity.
- The Pi 3's low power draw makes it ideal for extended hotel stays where wall power is available but entertainment options are limited.
- Tailscale will be used to maintain an encrypted tunnel back to the home network for remote administration and content transfers.
