# Security

## Philosophy

The homelab follows a **defense-in-depth** approach: multiple overlapping security layers at the network edge, host level, application level, and data level. The fundamental principle is **no trust in the WAN** — all external access is mediated through encrypted tunnels, and no services are exposed to the public internet.

---

## Edge Firewall: OPNsense

**OPNsense** serves as the primary edge firewall and router for the homelab.

- **Stateful inspection**: Tracks connection state; only return traffic for established connections is permitted.
- **Default-deny on WAN**: All inbound traffic from the internet is blocked unless explicitly permitted by a rule.
- **Outbound NAT**: Devices on the LAN share the public IP via NAT.
- **Additional services**: DHCP, DNS forwarding (upstream to Pi-hole), and the WireGuard fallback VPN.

No ports are forwarded to any internal host from the WAN interface.

---

## Host-Level Firewalls: UFW

Each Linux host runs **UFW (Uncomplicated Firewall)** as a second layer of defense:

### alphapi5 (Raspberry Pi 5)

Default policy: deny incoming, allow outgoing. Only necessary services are permitted:

| Service       | Port        | Source                |
|---------------|-------------|-----------------------|
| SSH           | 22/TCP      | Tailscale subnet      |
| Tailscale     | 41641/UDP   | Any (mesh traffic)    |

### Mac Mini

Similarly configured with UFW, allowing only essential services from the local LAN and Tailscale interfaces.

UFW rules are audited periodically to ensure no unintended exposures.

---

## DNS Filtering: Pi-hole

**Pi-hole** runs on the network as the primary DNS resolver, performing:

- **Ad and tracker blocking**: Queries to known advertising, tracking, and malware domains are blackholed.
- **DNS-level filtering only**: Does not inspect or block content beyond DNS — HTTPS traffic remains encrypted.
- **Upstream DNS**: Configured to use encrypted DNS resolvers (e.g., Cloudflare or Quad9 over TLS).

All LAN devices use Pi-hole as their DNS server (assigned via OPNsense DHCP).

---

## SSH Hardening

SSH is configured on all Linux hosts with the following settings:

- **Key-based authentication only**: Password authentication is disabled.
- **No root login**: Direct root SSH access is prohibited (`PermitRootLogin no`).
- **Allowed users**: Only specific user accounts may connect via SSH.
- **Listen interface**: SSH binds only to the internal network interface, not the WAN.

SSH keys are Ed25519, generated per-device with passphrase-protected private keys.

---

## API Tokens: Least-Privilege Access

### Proxmox

API tokens are created per-service with **scoped permissions**:

- Tokens are restricted to the minimum set of privileges required (e.g., read-only for monitoring, specific role-based access for automation).
- Tokens are revoked and rotated if no longer needed.

### Home Assistant

Long-lived access tokens are used for integrations and automation:

- Each token is scoped to the minimum required permissions.
- Tokens are stored securely in environment variables, not in configuration files.

---

## Remote Traffic Encryption

All traffic traversing the public internet is encrypted:

| Method        | Encryption            | Notes                          |
|---------------|-----------------------|--------------------------------|
| Tailscale     | WireGuard (Noise)     | End-to-end encrypted mesh      |
| WireGuard     | WireGuard (Noise)     | Fallback VPN (OPNsense)        |

Tailsec's mesh VPN ensures that even traffic between devices on different networks is encrypted without exposing any ports.

---

## Zero Exposed Ports

No services are directly exposed to the WAN. The homelab's attack surface visible from the internet is effectively **zero**:

- No HTTP/HTTPS ports forwarded to internal services.
- No SSH exposed to the internet.
- No game servers, media servers, or other services listening on the WAN interface.

All remote access is achieved through **outbound-only** VPN connections (Tailscale), meaning no inbound firewall rules are required.

---

## Secrets Management

Sensitive credentials and tokens are stored in a central environment file:

```
~/.hermes/.env
```

- API tokens (Proxmox, Home Assistant, etc.)
- Database passwords
- VPN keys
- Third-party service credentials

**Rules**:

- Secrets are **never** hardcoded in configuration files, scripts, or repositories.
- The `.env` file is **not** committed to version control.
- Applications source secrets from environment variables at runtime.
- File permissions on `.env` are restricted to the owning user (`chmod 600`).

---

## Future Improvements

The following security enhancements are planned:

| Improvement              | Description                                          |
|--------------------------|------------------------------------------------------|
| **VLANs**                | Segment the network into trusted, IoT, and guest VLANs to limit lateral movement. |
| **fail2ban**             | Install on Linux hosts to rate-limit and ban repeated failed SSH/login attempts. |
| **CrowdSec**             | Deploy CrowdSec on OPNsense and/or hosts for collaborative threat detection and blocking. |
| **Automatic Updates**    | Enable unattended-upgrades on Linux hosts to apply security patches promptly. |
| **Audit Logging**        | Centralize syslog and audit logs for intrusion detection and post-incident analysis. |
| **MFA for Dashboards**   | Add multi-factor authentication for Proxmox and Home Assistant web UIs. |

---

## Security Checklist

- [x] Edge firewall (OPNsense) — default-deny on WAN
- [x] Host firewalls (UFW) on all Linux machines
- [x] DNS filtering (Pi-hole)
- [x] Key-based SSH only, no passwords
- [x] Least-privilege API tokens
- [x] Encrypted remote access (Tailscale)
- [x] Zero ports exposed to WAN
- [x] Secrets in `~/.hermes/.env`, not in config files
- [ ] VLAN segmentation
- [ ] fail2ban / CrowdSec
- [ ] Automatic security updates
