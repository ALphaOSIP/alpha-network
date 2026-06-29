# Mac Mini (Late 2014) — Proxmox Server

## Hardware Specs

| Component | Detail |
|-----------|--------|
| **Model** | Mac Mini (Late 2014) — A1347 |
| **CPU** | Intel Core i5-4260U @ 1.4 GHz (2 cores, 4 threads) |
| **RAM** | 8 GB DDR3 |
| **Storage** | 256 GB Apple SSD |
| **Form Factor** | Headless (no display, no keyboard/mouse) |

## Software

| Component | Version / Detail |
|-----------|------------------|
| **Hypervisor** | Proxmox VE 9.1.1 |
| **Uptime** | 57+ days |
| **SSH Auth** | Key-based only (password auth disabled) |

## Network

| Interface | Address |
|-----------|---------|
| **Static IP (LAN)** | `10.0.1.108/24` |
| **Tailscale IP** | `100.124.155.110` |
| **Tailscale Subnet Router** | Advertises `10.0.1.0/24` |

---

## VMs & LXCs

### VM 100 — Home Assistant OS

- **OS**: Home Assistant OS 2026.6.4 (upgraded from 2026.4.3)
- **Resources**: 4 GB RAM, 32 GB disk
- **Web UI**: http://10.0.1.154:8123
- **Snapshot**: `pre-ha-setup` — clean state before MQTT/Zigbee/Tuya configuration
- **Rollback note (June 25):** VM was rolled back to `pre-ha-setup` after `configuration.yaml` modifications caused a boot loop. MQTT must be configured via the HA UI in 2026.x.

### LXC 101 — Omada Controller (Docker)

- **Software**: Omada Software Controller v5.15.x (via Docker)
- **Web UI**: http://10.0.1.141:8088

---

## Proxmox Setup Notes

### Initial Installation

1. Download the **Proxmox VE 9.x ISO** and write it to a USB drive (e.g., using `dd` or Balena Etcher).
2. Boot the Mac Mini from the USB (hold **Option/Alt** key during startup to select the boot device).
3. Install Proxmox onto the internal 256 GB Apple SSD following the on-screen prompts.
4. After first boot, configure networking:
   - Set static IP `10.0.1.108/24` with gateway `10.0.1.1`.
   - Ensure DNS is reachable (e.g., `10.0.1.1` or `1.1.1.1`).

### Post-Install Hardening

1. **Update packages:**
   ```bash
   apt update && apt upgrade -y
   ```

2. **Remove subscription nag** (optional — community/no-subscription repo only):
   ```bash
   sed -Ezi.bak "s/extjs\.examples\./console\.log('no thanks');/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
   ```

3. **Disable enterprise repos** and enable the free `pve-no-subscription` repository in `/etc/apt/sources.list`.

4. **Set up unattended-upgrades** for security patches.

---

## SSH Key Setup

Key-based SSH access is configured on the host. To replicate or verify:

1. Generate a key pair on the client (if not already present):
   ```bash
   ssh-keygen -t ed25519 -a 100
   ```

2. Copy the public key to the Mac Mini:
   ```bash
   ssh-copy-id -i ~/.ssh/id_ed25519.pub root@10.0.1.108
   ```

3. Verify that `/etc/ssh/sshd_config` contains:
   ```
   PermitRootLogin prohibit-password
   PubkeyAuthentication yes
   PasswordAuthentication no
   ```

4. Restart SSH if changes were made:
   ```bash
   systemctl restart sshd
   ```

---

## Tailscale Subnet Router Configuration

Tailscale is configured as a **subnet router** so devices on the Tailnet can reach the `10.0.1.0/24` LAN.

### Initial Setup

1. Install Tailscale:
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   ```

2. Authenticate and advertise the subnet:
   ```bash
   tailscale up --advertise-routes=10.0.1.0/24 --accept-routes=false
   ```

3. The Tailscale IP `100.124.155.110` is assigned automatically.

### Key Configuration Notes

- **`--accept-routes=false`**: The Mac Mini does not accept routes from other subnet routers.
- **`--advertise-routes=10.0.1.0/24`**: Advertises the local LAN so remote Tailscale nodes can reach devices like Home Assistant (`10.0.1.154`) and the Omada Controller (`10.0.1.141`).
- **Approval required**: In the [Tailscale Admin Console](https://login.tailscale.com), the advertised route must be approved (check the **Subnets** tab for the Mac Mini node).

### Persistent Service

Tailscale runs as a systemd service:
```bash
systemctl enable --now tailscaled
```

Check status:
```bash
tailscale status
tailscale ip
```

---

## Mounting Home Assistant OS Disk for Maintenance

Home Assistant OS runs inside a QEMU/KVM VM (VM 100). Its disk is a raw or qcow2 image located in the Proxmox storage.

### Locate the Disk

```bash
# Find the disk image for VM 100
find /var/lib/vz/images/100/ -type f
```

Typical path: `/var/lib/vz/images/100/vm-100-disk-0.qcow2` or `.raw`.

### Mount Procedure

1. **Stop the VM** (cannot mount while running):
   ```bash
   qm stop 100
   ```

2. **Load the network block device (NBD) kernel module:**
   ```bash
   modprobe nbd max_part=8
   ```

3. **Connect the QEMU disk image as a network block device:**
   ```bash
   qemu-nbd --connect=/dev/nbd0 /var/lib/vz/images/100/vm-100-disk-0.qcow2
   ```

4. **Find the partitions:**
   ```bash
   lsblk /dev/nbd0
   ```
   Home Assistant OS partitions typically include:
   - `nbd0p2` — boot (FAT32)
   - `nbd0p3` — main data partition (ext4)

5. **Mount the desired partition:**
   ```bash
   mkdir -p /mnt/ha-root
   mount /dev/nbd0p3 /mnt/ha-root
   ```

6. **Make changes** (e.g., edit `configuration.yaml`, inspect logs, restore backups).

7. **Unmount and disconnect:**
   ```bash
   umount /mnt/ha-root
   qemu-nbd --disconnect /dev/nbd0
   ```

8. **Start the VM again:**
   ```bash
   qm start 100
   ```

> **Warning**: Always stop the VM before mounting its disk. Mounting a running VM's disk will corrupt the filesystem.

---

## Creating Proxmox API Tokens

API tokens allow external tools (Terraform, Ansible, custom scripts) to authenticate with the Proxmox API.

### Via Web UI

1. Navigate to **Datacenter → Permissions → API Tokens**.
2. Click **Add**.
3. Select the user (e.g., `root@pam` or a dedicated user like `automation@pam`).
4. Set a **Token ID** (e.g., `terraform`, `ansible`, `hermes`).
5. (Optional) Set an expiration date and privilege separation.
6. Click **Add** — **copy the secret immediately**, it is only shown once.

### Via CLI

```bash
pveum user add automation@pam
pveum acl modify / --user automation@pam --role Administrator
pveum token add automation@pam --token-id my-token --privsep 1
```

Output includes the secret token — save it securely.

### Using the Token

```bash
curl -k -H "Authorization: PVEAPIToken=automation@pam!my-token=SECRET" \
  https://10.0.1.108:8006/api2/json/nodes
```

Store the token in your password manager or environment variables — never commit it to version control.

---

## Backup Strategy Notes

- Proxmox built-in backup (`vzdump`) can be scheduled under **Datacenter → Backup**.
- For VM 100 (Home Assistant), consider using the **Home Assistant Backup** feature to create snapshots that are stored on a separate NAS or cloud target.
- LXC 101 (Omada Controller) can be backed up via `vzdump` or by backing up the Docker volume/container.

---

## Troubleshooting

| Issue | Diagnosis / Fix |
|-------|-----------------|
| No video output (headless) | Normal — the Mac Mini runs without a display. Use SSH or the Proxmox Web UI (`https://10.0.1.108:8006`). |
| High CPU / load from Tailscale | Check `tailscale status` for active connections. Ensure no subnet routing loops. |
| VM won't start | Check `qm status <VM_ID>` and `cat /var/log/pve/tasks/active`. Verify disk space with `df -h`. |
| Can't reach LAN from Tailnet | Verify the subnet route is **approved** in the Tailscale admin console. Check `tailscale status --routes` on the Mac Mini. |
| SSH connection refused | Ensure `ssh` service is running (`systemctl status sshd`) and firewall allows port 22 (`iptables -L`). |
