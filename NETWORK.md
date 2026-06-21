# Alpha Network

## Overview

The **alpha-network** is a homelab network operating on the `10.0.1.0/24` subnet. It serves as the backbone for self-hosted services, media streaming, home automation, and lab experimentation.

**Subnet:** `10.0.1.0/24`  
**Default Gateway:** `10.0.1.1` (OPNsense)  
**DHCP:** OPNsense  
**DNS:** Pi-hole (`10.0.1.253`)  
**Public IP:** Dynamic (assigned by ISP via fiber ONT)

---

## Physical Topology

```
              Internet
                 │
            [ONT - Fiber Modem]
                 │
         [OPNsense - Dell OptiPlex]
          │                   │
    10.0.1.0/24 LAN         WAN (ISP)
          │
    [TL-SG108E Switch]
     /     │      │     \
    │      │      │      │
  Pi 5  Mac Mini  EAP 670  Pi 3
 .100   .108     (WiFi)   .101
   │
   ├── Pi-hole .253 (macvlan DNS)
   ├── RomM :3000
   ├── Jellyfin :8096
   ├── Uptime Kuma :3001
   ├── Immich :2283
   └── Portainer :9000
```

---

## Hardware & Roles

| Device | Model / Spec | Role |
|--------|-------------|------|
| **OPNsense** | Dell OptiPlex | Router, DHCP server, stateful firewall, WireGuard endpoint |
| **alphapi5** | Raspberry Pi 5 | Primary Docker server (25+ containers), Pi-hole host |
| **alphapi3** | Raspberry Pi 3 | Secondary node (currently offline) |
| **alphamox** | Mac Mini | Proxmox hypervisor, Omada Controller (LXC), HA VM |
| **TL-SG108E** | TP-Link 8-port | Unmanaged Gigabit switch (no VLANs configured) |
| **EAP 670** | TP-Link Omada | WiFi 6 access point (PoE powered) |

---

## IP Assignments

| Device | IP | Purpose |
|--------|-----|---------|
| OPNsense | `10.0.1.1` | Router, DHCP, firewall |
| alphapi5 | `10.0.1.100` | Primary Docker server |
| alphapi3 | `10.0.1.101` | Secondary node (offline) |
| HP Printer 1 | `10.0.1.103` | Office printer |
| alphamox | `10.0.1.108` | Proxmox hypervisor |
| HP Printer 2 | `10.0.1.116` | Secondary printer |
| Omada LXC | `10.0.1.141` | Omada SDN WiFi controller |
| LG TV | `10.0.1.143` | Smart TV |
| HA VM | `10.0.1.154` | Home Assistant |
| Eufy HomeBase | `10.0.1.187` | Camera bridge |
| Pi-hole | `10.0.1.253` | DNS resolver (macvlan container on Pi 5) |
| EAP 670 | `10.0.1.x` | WiFi access point (DHCP-assigned) |

> **Note:** The EAP 670 receives its address via DHCP — check OPNsense lease table or Omada Controller for current IP.

---

## DHCP

- **Server:** OPNsense (ISC DHCP or Kea, depending on OPNsense version)
- **Scope:** `10.0.1.50` – `10.0.1.254`
- **Reservations:** All static IPs above are configured as DHCP reservations by MAC address
- **Lease time:** 24 hours (default)
- **DNS servers advertised:** `10.0.1.253` (Pi-hole) — single server handed out to force all DNS through Pi-hole

### Adding a new reservation
1. Log into OPNsense → **Services > DHCP Server > LAN**
2. Scroll to **DHCP Static Mappings**
3. Click **+** — enter MAC address, IP address, hostname
4. Optionally set a static ARP entry for extra protection against IP spoofing

---

## DNS

- **Server:** Pi-hole v6 (containerized on alphapi5)
- **Address:** `10.0.1.253`
- **Network mode:** macvlan — has its own IP on the LAN, bypasses host networking
- **Upstream DNS:** Cloudflare (`1.1.1.1`, `1.0.0.1`) or Quad9 (`9.9.9.9`) — set via Pi-hole admin UI
- **Local domain:** `alpha.lan` (optional; conditional forwarding not currently configured)
- **Admin UI:** `http://10.0.1.253/admin` (password in vault)

All LAN clients receive Pi-hole as their only DNS server via DHCP. This ensures ad/tracker blocking applies network-wide. For devices that bypass DHCP (e.g., static IPs), configure DNS manually to `10.0.1.253`.

### Adding local DNS records
Pi-hole can be used for local name resolution via **Local DNS Records** in the admin panel. Alternatively, use OPNsense's **Unbound DNS** with **Host Overrides** if you prefer the router handle it. Currently, the network relies on IP addresses rather than hostnames for internal access.

---

## Firewall

### Edge (OPNsense)
- **Stateful inspection** enabled on all interfaces
- **Default policy:** Block inbound from WAN, allow all outbound
- **Anti-lockout rule:** Enabled (port 22/443 on LAN always accessible)
- **NAT:** Outbound NAT (automatic) for LAN → WAN traffic
- **Port forwards:** None on standard ports. Any external access goes through Tailscale.

### Host-level (UFW)

**alphapi5 (Pi 5)**
```
Status: active
To                         Action      From
--                         ------      ----
22/tcp (SSH)               ALLOW       10.0.1.0/24
9000/tcp (Portainer)       ALLOW       10.0.1.0/24
3000/tcp (RomM)            ALLOW       10.0.1.0/24
8096/tcp (Jellyfin)        ALLOW       10.0.1.0/24
3001/tcp (Uptime Kuma)     ALLOW       10.0.1.0/24
2283/tcp (Immich)          ALLOW       10.0.1.0/24
Anywhere                   ALLOW       Tailscale (100.x.y.z)
```

**alphamox (Mac Mini / Proxmox)**
```
Status: active
To                         Action      From
--                         ------      ----
22/tcp (SSH)               ALLOW       10.0.1.0/24
8006/tcp (Proxmox Web UI)  ALLOW       10.0.1.0/24
Anywhere                   ALLOW       Tailscale (100.x.y.z)
```

---

## Remote Access

### Tailscale (Primary)

Tailscale is the primary remote access method — a WireGuard-based mesh VPN with automatic NAT traversal.

| Node | Hostname | Tailscale IP |
|------|----------|-------------|
| alphapi5 | alphapi5 | `100.101.94.73` |
| alphamox | alphamox | `100.124.155.110` |
| iPhone | — | `100.89.238.113` |

**Subnet routing:** The Mac Mini (alphamox) is configured as a **subnet router** for `10.0.1.0/24`. This allows remote devices (e.g., iPhone) to reach LAN-only services like Home Assistant (`10.0.1.154:8123`) without exposing them to the internet.

**Setup summary:**
1. Install Tailscale on each node via `curl -fsSL https://tailscale.com/install.sh | sh`
2. Authenticate each node: `tailscale up`
3. On the subnet router: `tailscale up --advertise-routes=10.0.1.0/24 --accept-routes`
4. Approve the advertised routes in the [Tailscale Admin Console](https://login.tailscale.com) under **Machines > Edit Route Settings**

> **⚠️ Important routing fix:** On nodes that are *not* the subnet router (e.g., alphapi5), set `--accept-routes=false` to prevent Tailscale from routing local subnet traffic through the mesh tunnel instead of the LAN. Without this, traffic to `10.0.1.x` destinations gets sent through `tailscale0` instead of `eth0`, breaking local communication. This was a hard-learned lesson — the Pi 5 became unreachable from the LAN because all return traffic was going out through the Tailscale tunnel.

### WireGuard (Fallback)

WireGuard is configured on OPNsense but is **not actively used**. Tailscale replaced it for all remote access — no need to maintain port forwards or dynamic DNS when using Tailscale.

---

## WiFi

- **Access Point:** TP-Link EAP 670 (WiFi 6, 2.4/5 GHz)
- **Power:** PoE (via included PoE injector or PoE switch — currently via injector)
- **Management:** Omada SDN Controller
- **Controller address:** `http://10.0.1.141:8088`
- **Controller host:** LXC container on alphamox (Proxmox)

### SSIDs

| SSID | Band | Purpose |
|------|------|---------|
| `Alpha` | 2.4/5 GHz | Main network, bridged to LAN, clients get DHCP from OPNsense |
| `Alpha-IoT` | 2.4 GHz | IoT devices (cameras, smart plugs, printers) — isolated if VLANs are configured |

> Currently no VLANs are configured — both SSIDs bridge to the same flat `10.0.1.0/24` network.

---

## Switch

- **Model:** TP-Link TL-SG108E (8-port Gigabit)
- **Mode:** Unmanaged (operating as a simple dumb switch)
- **VLANs:** Not configured
- **Connections:**
  - Port 1: OPNsense LAN (uplink)
  - Port 2: alphapi5
  - Port 3: alphamox (Proxmox)
  - Port 4: EAP 670 (PoE injector)
  - Port 5: alphapi3 (offline)
  - Port 6–8: Open / future expansion

The "Easy Smart" features of this switch are unused; it functions purely as an unmanaged gigabit switch.

---

## Key Services (alphapi5)

The Raspberry Pi 5 (`10.0.1.100`) runs Docker with the following publicly accessible LAN services:

| Service | URL | Port |
|---------|-----|------|
| Portainer | `http://10.0.1.100:9000` | Docker management |
| Jellyfin | `http://10.0.1.100:8096` | Media streaming |
| RomM | `http://10.0.1.100:3000` | ROM manager |
| Uptime Kuma | `http://10.0.1.100:3001` | Uptime monitoring |
| Immich | `http://10.0.1.100:2283` | Photo backup |

---

## Maintenance Notes

### Rebooting OPNsense
- SSH to `10.0.1.1` or use the web UI at `https://10.0.1.1`
- Command: `sudo reboot`
- Expect ~2 min downtime during boot — all LAN traffic halts until the firewall is back up

### Pi-hole updates
- Run inside the Docker container: `docker exec pihole pihole -up`
- Gravity (blocklist) update: `docker exec pihole pihole updateGravity`
- Web UI: `http://10.0.1.253/admin`

### Adding a new device to the network
1. Physically connect (Ethernet) or connect to the `Alpha` SSID (WiFi)
2. Device gets a DHCP lease in the `10.0.1.50–254` range
3. Optionally create a **DHCP reservation** in OPNsense for a static IP
4. Point DNS to `10.0.1.253` manually if the device uses static networking

### Troubleshooting connectivity
1. **Is the device on the right network?** Check that WiFi SSID is `Alpha` (not guest)
2. **Does it have an IP?** Run `ip a` (Linux) or `ipconfig` (Windows) — should be `10.0.1.x`
3. **Can it reach the gateway?** `ping 10.0.1.1`
4. **DNS working?** `nslookup google.com 10.0.1.253` — should resolve
5. **Check Pi-hole query log** at `http://10.0.1.253/admin/query_log.php` — look for blocked queries
6. **Check OPNsense firewall logs** at **Firewall > Log Files**

---

## Future Improvements

- [ ] Configure **VLANs** for IoT device isolation
- [ ] Enable **mDNS reflector** on OPNsense for service discovery across subnets
- [ ] Set up **conditional forwarding** in Pi-hole for `alpha.lan` domain resolution
- [ ] Replace flat switch with a **managed PoE switch** (e.g., TL-SG2008P) for VLAN trunking
- [ ] Implement **automatic backup** of OPNsense config to alphapi5
- [ ] Add a **fallback internet connection** (4G LTE failover via USB modem on OPNsense)
