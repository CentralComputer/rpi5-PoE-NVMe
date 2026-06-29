# Build a Bulletproof Edge Node with Raspberry Pi 5, Native NVMe, and Zero-Trust Remote Access

*By Richard Rives*

*A practical guide to deploying a PoE+-powered, remotely manageable Pi 5 that boots from NVMe over native PCIe and connects securely through Tailscale and Teleport.*

---

## Why This Build?

The Raspberry Pi 5 is the first Pi with a native PCIe interface — and that changes everything for edge deployments. No more USB-to-NVMe adapters. No more SD card reliability headaches. Just a direct PCIe Gen 2 connection to a fast NVMe drive, real NVMe speeds (~900 MB/s read), and a platform powerful enough to run serious workloads at the edge.

Pair that with a combined PoE+ and NVMe HAT (Geekworm X1012), Tailscale, Teleport, and Atera RMM, and you get a single-cable edge node — one Ethernet wire for both power and network — that any authorized technician can reach from anywhere, with full session auditing and remote monitoring built in.

This build is designed to be replicated identically across multiple locations. Same parts, same steps, same result every time.

---

## Why Pi 5 Over Pi 4?

The Pi 5 is the right choice here for three reasons:

**Native PCIe.** The Pi 4 had no PCIe port, so NVMe storage required a USB 3.0-to-M.2 adapter — capped at ~500 MB/s and one more component to source and fail. The Pi 5 exposes PCIe Gen 2 x1 via a flat FPC connector, and the HAT connects directly to it. The NVMe drive shows up as `/dev/nvme0n1` just like it would in a server.

**More power, more headroom.** The Pi 5's quad-core Cortex-A76 CPU is roughly 2–3x faster than the Pi 4's A72. For edge compute, monitoring, and AI inference workloads, that margin matters.

**Cleaner build.** One combined HAT (Geekworm X1012) handles PoE+ power delivery and NVMe storage. Fewer parts, fewer failure points, one FPC cable instead of a USB enclosure dangling off a port.

---

## The Parts List

| Component | Details |
|---|---|
| Raspberry Pi 5 — 8 GB RAM | Main compute unit; native PCIe Gen 2 via FPC connector. [centralcomputer.com](https://www.centralcomputer.com/raspberry-pi-5-8gb-ram-board.html) — $174.99 |
| Geekworm X1012 PoE+ HAT and NVMe SSD Shield | Combined PoE+ (802.3at, 25W, 5V/5A) + M.2 NVMe HAT; ~$39.95. [centralcomputer.com](https://www.centralcomputer.com/geekworm-x1012-pcie-to-nvme-poe-shield-for-raspberry-pi-5.html) |
| Geekworm P579 Metal Case for Raspberry Pi 5 | PCIe HAT-compatible enclosure, confirmed fit with X1012. [centralcomputer.com](https://www.centralcomputer.com/geekworm-p579-raspberry-pi-5-pcie-metal-case-black.html) — $12.95 |
| SanDisk High Endurance 128 GB microSDXC (SDSQQNR-128G) | Bootstrap and recovery medium; High Endurance rated for continuous writes. [centralcomputer.com](https://www.centralcomputer.com/sandisk-sdsqqnr-128g-an6ia-128gb-high-enduranceuhs-i-microsdxc-memory-card-with-sd-adapter.html) — $39.99 |
| Western Digital SN770M 500 GB NVMe M.2 2230 SSD | PCIe Gen4 x4, 5150 MB/s read; primary OS and data drive. [centralcomputer.com](https://www.centralcomputer.com/western-digital-sn770m-500gb-nvme-m-2-2230ssd-m-2-2230-pcie-gen4-x4-5150mb-s-reads-4900mb-s-writes.html) — $129.99 |
| HDMI-to-CSI-2 Adapter | Captures HDMI video input via camera ribbon slot (optional) |
| Cat6 Ethernet Cable | Single cable delivers both network and PoE+ power. [centralcomputer.com](https://www.centralcomputer.com/all-products/cables/networking-cables/ethernet-cables/cat6.html) — length per deployment |
| Ubiquiti USW-Flex-2.5G-8-PoE Switch (optional) | 8-port 802.3at PoE+ switch with 2.5G ports and 10GbE uplink — required if deploying without an existing PoE+ switch. [centralcomputer.com](https://www.centralcomputer.com/ubiquiti-usw-flex-2-5g-8-poe-flexible-2-5g-8-port-poe-switch-with-a-10-gbe-rj45-sfp-combination-uplink-port-that-can-be-powere.html) — $209.99 |

> **One HAT, two jobs.** The Geekworm X1012 replaces two components from a Pi 4 build: the separate PoE HAT and the USB NVMe enclosure. It connects to the Pi 5's PCIe FPC port via the included 25mm flat cable and to the GPIO header for power management. The result is a cleaner, more reliable build with true NVMe performance.

> **Case note:** The Geekworm P579 Metal Case is confirmed compatible with the X1012 HAT. It is available at [centralcomputer.com](https://www.centralcomputer.com/geekworm-p579-raspberry-pi-5-pcie-metal-case-black.html) for $12.95.

---

## The Operating System: Ubuntu 24.04 LTS

Ubuntu 24.04.2 LTS (Noble Numbat) is the right OS for this build — full Raspberry Pi 5 support, supported until April 2029.

**Download links:**

- [Ubuntu 24.04.2 LTS Server for Raspberry Pi (recommended for headless deployments)](https://cdimage.ubuntu.com/releases/24.04/release/ubuntu-24.04.2-preinstalled-server-arm64+raspi.img.xz)
- [Ubuntu 24.04.2 LTS Desktop for Raspberry Pi](https://cdimage.ubuntu.com/releases/24.04/release/ubuntu-24.04.2-preinstalled-desktop-arm64+raspi.img.xz)
- [Raspberry Pi Imager (flash tool)](https://www.raspberrypi.com/software/)

Use the Server image for headless deployments. Set the hostname, enable SSH, and configure credentials in Raspberry Pi Imager's Advanced Options before writing — no monitor or keyboard needed after that.

---

## Hardware Assembly

1. Insert the NVMe SSD into the M-Key slot on the Geekworm X1012 and secure with the M.2 standoff and screw.
2. Connect the 25mm PCIe FPC cable (included) between the HAT's PCIe port and the Pi 5's PCIe FPC connector — lift the latch, insert gold-contacts-down, press the latch closed on both ends.
3. Seat the HAT's 40-pin GPIO header extension onto the Pi 5. Press firmly and secure the standoffs.
4. Install the assembly into your Pi 5 case.
5. Insert the SanDisk microSD card and connect the Ethernet cable to a PoE+ (802.3at) switch port.

Power on only after OS is flashed.

---

## Flashing and First Boot

Flash Ubuntu 24.04.2 LTS Server to the SanDisk microSD using Raspberry Pi Imager. Set hostname, SSH credentials, and locale in Advanced Options before writing.

After first boot, SSH in and update:

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

---

## Migrating to NVMe

On the Pi 5, the NVMe drive appears immediately as `/dev/nvme0n1` — no USB adapter, no special drivers.

**Update the bootloader and set boot order:**

```bash
sudo rpi-eeprom-update -a
sudo reboot

# Set NVMe as first boot priority
sudo -E rpi-eeprom-config --edit
# Set: BOOT_ORDER=0xf416  (NVMe first, then SD, then USB, then network)
```

**Clone the SD card to NVMe:**

```bash
lsblk  # confirm nvme0n1 is visible
sudo apt install rpi-clone -y
sudo rpi-clone nvme0n1
sudo reboot

# Verify after reboot
findmnt / | grep nvme
```

Remove the SD card, label it with the hostname and build date, and store it inside the case as a recovery fallback.

---

## Power: One Cable, No Bricks

The Geekworm X1012 draws power from an 802.3at PoE+ switch port — up to 25W, which is exactly what the Pi 5 needs at full load. Connect a single Cat6 cable to a PoE+ switch and the node is both powered and networked.

> **Use PoE+ (802.3at), not standard PoE (802.3af).** The Pi 5 requires 5V/5A (25W). Standard PoE only delivers 15W and will cause instability under load. Confirm your switch ports are PoE+ rated before deployment. See the official [Raspberry Pi USB Power Delivery white paper](https://pip-assets.raspberrypi.com/categories/685-app-notes-guides-whitepapers/documents/RP-009856-WP-1-USB%20Power%20delivery%20on%20Raspberry%20Pi%205.pdf) for full details.

For bench setup before a PoE switch is available, use a 27W USB-C power supply (Raspberry Pi official or equivalent).

---

## Remote Access: Tailscale

[Tailscale](https://tailscale.com) creates a private WireGuard mesh between all your devices. Once a Pi joins your tailnet, SSH access is available from anywhere — no open firewall ports.

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --authkey=<your-auth-key> --hostname=pi-store-01
sudo systemctl enable tailscaled
```

Generate auth keys per-device from the [Tailscale admin console](https://login.tailscale.com/admin/settings/keys) for unattended rollouts.

---

## Remote Access: Teleport (Open Source)

[Teleport](https://goteleport.com) adds certificate-based SSH, session recording, audit logging, and role-based access control across the fleet.

```bash
# Install Teleport node agent
curl https://goteleport.com/static/install.sh | bash -s 16 oss

# Register with your Teleport cluster
sudo teleport node configure \
  --auth-server=teleport.yourcompany.com:443 \
  --token=<JOIN_TOKEN> \
  --labels=location=store-01,role=edge-node \
  --output /etc/teleport.yaml

sudo systemctl enable teleport && sudo systemctl start teleport
```

Nodes appear in `tsh ls` and sessions are recorded in the Teleport audit log. Use short-lived, per-device join tokens — never reuse them across nodes.

---

## Remote Monitoring: Atera RMM

[Atera](https://www.atera.com) handles monitoring, alerting, patch management, and helpdesk — and includes a built-in SSH terminal for Linux devices under **Devices > Manage > SSH**, so your team can open a terminal session directly from the Atera console without a separate SSH client.

Install the agent from the Atera dashboard (**Devices > Add Agent > Linux**) — each generated command is customer-specific:

```bash
# Paste the command generated in Atera for this customer
curl -s 'https://app.atera.com/api/v2/agents/installers/linux?...' -o AteraAgent.sh && bash AteraAgent.sh

sudo systemctl status atera-agent
```

---

## Verify NVMe Performance

One quick check worth running after migration to confirm you're getting native PCIe speeds:

```bash
sudo apt install -y nvme-cli
sudo nvme smart-log /dev/nvme0n1   # check health
sudo hdparm -t /dev/nvme0n1        # expect ~800-900 MB/s on Pi 5 PCIe Gen 2
```

If you're seeing USB 3.0 speeds (~400 MB/s), the FPC cable may not be latched correctly or the boot is still coming from SD.

---

## The Result

One cable. One device. Native NVMe at ~900 MB/s. No exposed ports. Remote access through both Tailscale and Teleport. SSH terminal and monitoring through Atera. A build that can be replicated across every store location with identical parts and identical steps.

The Pi 5 closes the gap between a hobbyist SBC and a real edge appliance — and this build makes the most of it.

---

## Resources

- [Geekworm X1012 for Raspberry Pi 5](https://pineboards.io/products/hatdrive-poe-for-raspberry-pi-5)
- [Ubuntu 24.04.2 LTS Download Page](https://ubuntu.com/download/raspberry-pi)
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- [Pi 5 NVMe Compatibility List (Jeff Geerling)](https://pipci.jeffgeerling.com)
- [Tailscale Download](https://tailscale.com/download/linux)
- [Teleport Open Source Download](https://goteleport.com/download/)
- [Atera RMM](https://www.atera.com)
- [rpi-clone (GitHub)](https://github.com/billw2/rpi-clone)
- [Raspberry Pi 5 PCIe / Boot Documentation](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#raspberry-pi-5)

---

*Questions about the Geekworm X1012 or NVMe drive compatibility? Drop a comment below.*
