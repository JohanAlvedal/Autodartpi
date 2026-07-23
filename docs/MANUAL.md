# AutoDarts Pi Image
## Technical Manual & Deployment Guide

> Headless Raspberry Pi setup with WiFi captive portal and automatic updates  
> Version 1.1.1 • Debian 13 (Trixie) 64-bit  
> By Johan Alvedal

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Requirements & Hardware](#2-requirements--hardware)
3. [Flashing the Image to SD Card](#3-flashing-the-image-to-sd-card)
   - 3.1 [Using Raspberry Pi Imager (recommended)](#31-using-raspberry-pi-imager-recommended)
   - 3.2 [Using balenaEtcher](#32-using-balenaetcher)
   - 3.3 [Using dd (Linux/Mac – advanced)](#33-using-dd-linuxmac--advanced)
4. [Pre-configuration via autodarts.conf](#4-pre-configuration-via-autodartconf)
5. [First Boot – Step by Step](#5-first-boot--step-by-step)
   - 5.1 [Boot sequence in detail](#51-boot-sequence-in-detail)
   - 5.2 [What happens in the background?](#52-what-happens-in-the-background)
6. [WiFi Setup via Captive Portal](#6-wifi-setup-via-captive-portal)
   - 6.1 [Connecting to AutoDart-Setup](#61-connecting-to-autodart-setup)
   - 6.2 [Filling in the portal](#62-filling-in-the-portal)
   - 6.3 [What happens after saving?](#63-what-happens-after-saving)
7. [Claiming the AutoDarts Board](#7-claiming-the-autodarts-board)
8. [Automatic Updates](#8-automatic-updates)
9. [SSH Access & CLI Management](#9-ssh-access--cli-management)
10. [System Files & Configuration](#10-system-files--configuration)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. System Overview

AutoDarts Pi Image is a pre-built Raspberry Pi installation that allows you to deploy AutoDarts hardware without a monitor, keyboard, or network cable. Network setup is completed wirelessly through a built-in captive portal when no working Ethernet or saved WiFi connection is available.

| Property | Value |
|----------|-------|
| Base OS | Debian 13 (Trixie) 64-bit |
| Target Hardware | Raspberry Pi 4 / Raspberry Pi 5 |
| SD Card (minimum) | 8 GB Class 10 / A1 |
| Hostname | `autodartpi.local` |
| WiFi setup SSID | `AutoDart-Setup` |
| Setup portal | http://192.168.4.1 |
| Board Manager | http://autodartpi.local:3180 |
| Board Manager HTTPS | https://autodartpi.local:3181 |
| SSH access | `ssh autodart@autodartpi.local` |
| Automatic updates | 5 minutes after boot, then every 3 days |

### Process Overview

| Step | Action |
|------|--------|
| 1 | Flash the `.img.xz` file to an SD card using Raspberry Pi Imager |
| 2 | *(Optional)* Pre-configure WiFi in `/boot/firmware/autodarts.conf` |
| 3 | Boot the Raspberry Pi and wait approximately 60–90 seconds |
| 4 | If no working network is available, connect to **AutoDart-Setup** |
| 5 | Select your WiFi network and enter its password in the setup portal |
| 6 | The Raspberry Pi connects to your WiFi and stops the setup access point |
| 7 | Open the Board Manager at **http://autodartpi.local:3180** |
| 8 | Claim the board in the AutoDarts Board Manager |
| 9 | AutoDarts checks for updates 5 minutes after boot and then every 3 days |

> **Note:** If Ethernet is connected or a saved WiFi connection is already working, `AutoDart-Setup` is not started.

---

## 2. Requirements & Hardware

### Required

- Raspberry Pi 4 Model B with 2 GB RAM or more, or Raspberry Pi 5
- MicroSD card, minimum 8 GB, recommended 16 GB Class 10 / A1 or A2
- USB-C power supply:
  - Raspberry Pi 4: 5V / 3A
  - Raspberry Pi 5: 5V / 5A recommended
- AutoDarts cameras and USB hub according to AutoDarts hardware requirements
- A computer with an SD card reader to flash the image
- A phone or computer with WiFi for initial network setup

### Recommended

- Raspberry Pi Imager installed on your computer
- A reliable Class 10, A1 or A2 microSD card
- Ethernet access during testing or troubleshooting
- A second microSD card for testing new image releases

> **Note:** The Raspberry Pi does not need to be connected to a monitor or keyboard. The entire normal setup process is headless.

---

## 3. Flashing the Image to SD Card

The image is distributed as a compressed `.img.xz` file. Raspberry Pi Imager and balenaEtcher can flash it directly without extracting it first.

### 3.1 Using Raspberry Pi Imager (recommended)

Raspberry Pi Imager is the easiest method and works on Windows, macOS and Linux.

| Step | Action |
|------|--------|
| 1 | Download and install Raspberry Pi Imager from [raspberrypi.com/software](https://www.raspberrypi.com/software/) |
| 2 | Insert the microSD card into your computer |
| 3 | Launch Raspberry Pi Imager |
| 4 | Select **Choose Device** and choose your Raspberry Pi model |
| 5 | Select **Choose OS** |
| 6 | Scroll down and select **Use custom** |
| 7 | Select `autodarts-pi-v1.1.1.img.xz` |
| 8 | Select **Choose Storage** and choose the microSD card |
| 9 | Click **Write** and confirm |
| 10 | Wait until writing and verification complete |
| 11 | Safely eject the microSD card and insert it into the Raspberry Pi |

> ⚠️ **Warning:** The selected storage device will be erased. Always verify that you selected the microSD card and not another disk.

> **Tip:** Raspberry Pi Imager verifies the flashed image automatically. If verification fails, try writing the image again or use another microSD card.

### 3.2 Using balenaEtcher

balenaEtcher is an alternative tool that works on Windows, macOS and Linux.

| Step | Action |
|------|--------|
| 1 | Download balenaEtcher from [etcher.balena.io](https://etcher.balena.io/) |
| 2 | Open balenaEtcher and click **Flash from file** |
| 3 | Select `autodarts-pi-v1.1.1.img.xz` |
| 4 | Select the microSD card under **Select target** |
| 5 | Click **Flash!** |
| 6 | Wait for writing and verification to complete |
| 7 | Safely eject the microSD card |

### 3.3 Using dd (Linux/Mac – advanced)

Advanced Linux and macOS users can decompress and flash the image using the command line.

Linux example:

```bash
xzcat autodarts-pi-v1.1.1.img.xz | sudo dd of=/dev/sdX bs=4M status=progress conv=fsync
```

Replace `/dev/sdX` with the actual microSD card device.

Verify the correct device before flashing:

```bash
lsblk
```

On macOS:

```bash
diskutil list
```

Unmount the card before writing:

```bash
diskutil unmountDisk /dev/diskX
```

Flash using the raw disk device:

```bash
xzcat autodarts-pi-v1.1.1.img.xz | sudo dd of=/dev/rdiskX bs=4m
```

> ⚠️ **Warning:** `dd` does not provide a confirmation dialog. Specifying the wrong output device can permanently erase another disk.

---

## 4. Pre-configuration via autodarts.conf

The captive portal can be skipped by pre-configuring the WiFi network before first boot.

Create or edit:

```text
/boot/firmware/autodarts.conf
```

Use this format:

```ini
WIFI_SSID="YourNetworkName"
WIFI_PASSWORD="YourPassword"
```

| Parameter | Description |
|-----------|-------------|
| `WIFI_SSID` | Name of the WiFi network. The value is case-sensitive. |
| `WIFI_PASSWORD` | Password for the WiFi network. |

Board ID and API key are not stored in this file. Board claiming is completed separately in the AutoDarts Board Manager.

> **Security note:** The WiFi password is stored as plain text in this file. Restrict physical access to the microSD card.

> **Note:** If the file is empty, missing, or the configured network is unavailable, the Raspberry Pi starts `AutoDart-Setup`.

---

## 5. First Boot – Step by Step

### 5.1 Boot sequence in detail

The exact timing depends on the Raspberry Pi model, microSD card and power supply.

| Approximate time | Event |
|------------------|-------|
| 0:00 | Power is connected and the Raspberry Pi begins booting |
| 0:05–0:20 | Bootloader and Linux kernel start |
| 0:15–0:45 | Debian starts and systemd launches services |
| 0:30–0:60 | `setup-ap.service` checks for an active Ethernet or saved WiFi connection |
| 0:30–0:90 | If a working connection exists, setup mode remains disabled |
| 0:30–0:90 | If no working connection exists, `AutoDart-Setup` and the portal are started |
| Up to 15 minutes | Setup mode remains available while waiting for WiFi configuration |
| After WiFi setup | The setup access point and portal stop |
| After connection | Board Manager becomes available on the normal network |

> **Note:** First boot can take longer while the filesystem and services initialize.

### 5.2 What happens in the background?

#### setup-ap.service

The script `/usr/local/bin/setup-ap.sh` manages the network setup process.

It performs the following checks and actions:

- Checks for an active Ethernet connection
- Checks for an active WiFi connection
- Excludes the setup network `AutoDart-Setup` when deciding whether normal WiFi is available
- Stops the captive portal and removes the setup access point when a normal connection is available
- Reads WiFi credentials from `/boot/firmware/autodarts.conf`, if configured
- Attempts to connect to the configured WiFi network
- Starts `AutoDart-Setup` only when no normal network connection is available
- Starts the captive portal while setup mode is active
- Keeps setup mode available for up to 15 minutes
- Periodically checks whether a working network connection has become available
- Stops the setup access point and portal after a successful connection

#### portal.service

The Flask application at `/home/autodart/portal/app.py` runs only while setup mode is active.

The portal allows the user to:

- Scan for nearby WiFi networks
- Select a WiFi network
- Enter its password
- Save the SSID and password to `/boot/firmware/autodarts.conf`
- Connect the Raspberry Pi using NetworkManager
- Display an error if the connection fails
- Stop the setup access point after a successful connection
- Continue setup through the AutoDarts Board Manager

The portal does not request or store an AutoDarts Board ID or API key.

> **Note:** NetworkManager can take some time to determine that a password is incorrect. A failed connection may therefore take approximately 30 seconds before an error appears.

#### autodarts.service

AutoDarts runs as a systemd service.

The runtime configuration is stored in:

```text
/home/autodart/.config/autodarts/config.toml
```

The release image is distributed without a claimed AutoDarts board identity. On a new installation, AutoDarts starts in an unclaimed state.

Open the Board Manager at:

```text
http://autodartpi.local:3180
```

Then follow the displayed instructions to claim the board.

#### autodarts-updater.service and autodarts-updater.timer

The updater is controlled by a systemd service and timer.

The timer triggers:

- 5 minutes after boot
- Every 3 days after the previous timer activation

The updater:

- Runs without user interaction
- Checks for and installs an available AutoDarts update
- Restarts AutoDarts as part of the update process
- Writes output to the systemd journal

---

## 6. WiFi Setup via Captive Portal

### 6.1 Connecting to AutoDart-Setup

When no working Ethernet or saved WiFi connection is available, the Raspberry Pi creates an open WiFi network.

| Property | Detail |
|----------|--------|
| SSID | `AutoDart-Setup` |
| Password | None |
| Raspberry Pi address | `192.168.4.1` |
| Client address | `192.168.4.x` |
| Setup time limit | Up to 15 minutes |

Connect your phone or computer to `AutoDart-Setup`.

Most phones and computers will detect the captive portal and open it automatically. If it does not open, navigate manually to:

```text
http://192.168.4.1
```

> **Tip:** On a phone, temporarily disable mobile data if the setup page does not appear. Some phones continue using the mobile network because `AutoDart-Setup` has no internet access.

> **Note:** The setup network is open and has no password. Use it only for initial local configuration.

### 6.2 Filling in the portal

The portal scans for nearby WiFi networks.

Complete the following steps:

1. Select your WiFi network from the list.
2. Enter the WiFi password.
3. Submit the form.
4. Wait while the Raspberry Pi attempts to connect.
5. If an error is displayed, verify the SSID and password and try again.
6. After a successful connection, reconnect your phone or computer to the normal WiFi network.
7. Open the AutoDarts Board Manager at `http://autodartpi.local:3180`.

The portal only handles WiFi configuration. Board claiming takes place in the Board Manager.

### 6.3 What happens after saving?

| Step | Action |
|------|--------|
| 1 | The form is sent to the Flask application using HTTP POST |
| 2 | `app.py` saves the SSID and password to `/boot/firmware/autodarts.conf` |
| 3 | NetworkManager attempts to connect to the selected WiFi network |
| 4 | If the connection succeeds, the setup access point is stopped |
| 5 | `portal.service` exits |
| 6 | The phone or computer loses its connection to `AutoDart-Setup` |
| 7 | The Raspberry Pi becomes available on the normal WiFi network |
| 8 | AutoDarts runs in its unclaimed state |
| 9 | The user opens Board Manager and claims the board |

> **Note:** Losing the connection to `AutoDart-Setup` after a successful setup is normal and expected.

---

## 7. Claiming the AutoDarts Board

After the Raspberry Pi is connected to the normal network, open:

```text
http://autodartpi.local:3180
```

The Board Manager displays the instructions required to claim the board.

Complete the board claim in the Board Manager instead of entering credentials in the captive portal or `autodarts.conf`.

If the hostname does not resolve, find the Raspberry Pi IP address in your router and open:

```text
http://PI_IP_ADDRESS:3180
```

For example:

```text
http://192.168.1.123:3180
```

The HTTPS interface is available on port 3181:

```text
https://autodartpi.local:3181
```

A browser may display a certificate warning for the local HTTPS connection. The normal HTTP interface on port 3180 can be used instead.

---

## 8. Automatic Updates

AutoDarts checks for updates automatically:

- 5 minutes after boot
- Every 3 days after that

No normal user action is required.

### Checking update status

View the updater log:

```bash
journalctl -u autodarts-updater.service -n 50
```

See when the next update is scheduled:

```bash
systemctl list-timers autodarts-updater.timer
```

Check timer status:

```bash
systemctl status autodarts-updater.timer
```

### Manual update

Start an update manually:

```bash
sudo systemctl start autodarts-updater.service
```

Follow the update log live:

```bash
journalctl -u autodarts-updater.service -f
```

> **Note:** AutoDarts can restart during an update. Any active game or detection session may be interrupted.

---

## 9. SSH Access & CLI Management

Once the Raspberry Pi is connected to the network, SSH can be used for administration and troubleshooting.

### Logging in

```bash
ssh autodart@autodartpi.local
```

### Default Credentials

| | |
|--|--|
| SSH user | `autodart` |
| SSH password | `autodart` |

> ⚠️ **Important:** Change the default password after the first SSH login:

```bash
passwd
```

The hostname `autodartpi.local` uses mDNS.

- macOS and most Linux systems support mDNS automatically.
- Modern Windows systems often resolve `.local` addresses, but support can vary.
- If the hostname does not work, use the Raspberry Pi IP address instead.

Example:

```bash
ssh autodart@192.168.1.123
```

### Quick reference

| Task | Command |
|------|---------|
| Log in through SSH | `ssh autodart@autodartpi.local` |
| Open Board Manager | `http://autodartpi.local:3180` |
| Check AutoDarts status | `systemctl status autodarts.service` |
| Restart AutoDarts | `sudo systemctl restart autodarts.service` |
| Stop AutoDarts | `sudo systemctl stop autodarts.service` |
| Start AutoDarts | `sudo systemctl start autodarts.service` |
| Trigger a manual update | `sudo systemctl start autodarts-updater.service` |
| Follow AutoDarts logs | `journalctl -u autodarts.service -f` |
| Follow updater logs | `journalctl -u autodarts-updater.service -f` |
| Check the update timer | `systemctl status autodarts-updater.timer` |
| Check the setup service | `systemctl status setup-ap.service` |
| View WiFi configuration | `sudo cat /boot/firmware/autodarts.conf` |
| Edit WiFi configuration | `sudo nano /boot/firmware/autodarts.conf` |
| Reboot | `sudo reboot` |
| Shut down | `sudo poweroff` |

### Checking service status

```bash
systemctl status autodarts.service
systemctl status autodarts-updater.service
systemctl status autodarts-updater.timer
systemctl status setup-ap.service
systemctl status portal.service
```

The portal service should normally be inactive after the Raspberry Pi has connected to the normal network.

### Useful logs

All logs from the current boot:

```bash
journalctl -b
```

AutoDarts logs:

```bash
journalctl -u autodarts.service --since today
```

WiFi setup logs:

```bash
journalctl -u setup-ap.service --since today
```

Captive portal logs:

```bash
journalctl -u portal.service --since today
```

Updater logs:

```bash
journalctl -u autodarts-updater.service -n 100
```

Live system log:

```bash
journalctl -f
```

---

## 10. System Files & Configuration

| File or path | Description |
|--------------|-------------|
| `/usr/local/bin/setup-ap.sh` | Main script for normal network detection, setup AP management and captive portal control |
| `/home/autodart/portal/app.py` | Flask captive portal for WiFi configuration |
| `/etc/systemd/system/setup-ap.service` | systemd service that launches the network setup logic |
| `/etc/systemd/system/portal.service` | Static systemd service started only while the captive portal is needed |
| `/etc/systemd/system/autodarts.service` | systemd service for AutoDarts |
| `/etc/systemd/system/autodarts-updater.service` | systemd service for updating AutoDarts |
| `/etc/systemd/system/autodarts-updater.timer` | Timer that runs the updater after boot and every 3 days |
| `/boot/firmware/autodarts.conf` | WiFi SSID and password pre-configuration |
| `/home/autodart/.config/autodarts/config.toml` | AutoDarts runtime configuration and board claim state |
| `/home/autodart/build-image.sh` | Image builder used to copy, shrink and compress the release image |

### Example autodarts.conf

```ini
WIFI_SSID="YourNetworkName"
WIFI_PASSWORD="YourPassword"
```

Do not add Board ID or API key values to this file.

### Checking the current network connection

```bash
nmcli device status
```

Show active connections:

```bash
nmcli connection show --active
```

Show available WiFi networks:

```bash
nmcli device wifi list
```

Show the Raspberry Pi IP addresses:

```bash
hostname -I
```

---

## 11. Troubleshooting

| Problem | Possible solution |
|---------|-------------------|
| `AutoDart-Setup` is not visible | Wait 60–90 seconds. Disconnect Ethernet and restart the Raspberry Pi. |
| `AutoDart-Setup` is still not visible | Check `systemctl status setup-ap.service` and `journalctl -u setup-ap.service -n 100`. |
| Portal does not open automatically | Open `http://192.168.4.1` manually. |
| Phone refuses to show the portal | Temporarily disable mobile data and reconnect to `AutoDart-Setup`. |
| Portal displays a WiFi error | Verify the selected SSID and password. A failed attempt can take approximately 30 seconds. |
| Raspberry Pi does not connect to WiFi | Verify the SSID, password, WiFi country setting and router channel. |
| Setup access point disappears after saving | This is expected after a successful WiFi connection. |
| Setup access point shuts down before configuration | Setup mode is available for up to 15 minutes. Restart the Raspberry Pi without Ethernet. |
| Portal cannot be opened from the normal LAN | This is expected. The portal only runs while `AutoDart-Setup` is active. |
| `autodartpi.local` cannot be reached | Find the Raspberry Pi IP address in the router and use the IP address directly. |
| Board Manager does not load | Check `systemctl status autodarts.service` and verify that port 3180 is reachable. |
| Board is not claimed | Open Board Manager and follow the claim instructions. |
| AutoDarts does not start | Run `systemctl status autodarts.service` and inspect the journal. |
| Automatic update does not run | Check `systemctl status autodarts-updater.timer` and `systemctl list-timers`. |
| SSH login fails | Verify the username, password and IP address. |
| WiFi configuration must be replaced | Edit `/boot/firmware/autodarts.conf`, remove old NetworkManager profiles if needed, and reboot. |

### Useful troubleshooting commands

Check the current IP address:

```bash
hostname -I
```

Check normal network connections:

```bash
nmcli device status
nmcli connection show --active
```

Check setup service:

```bash
systemctl status setup-ap.service --no-pager
journalctl -u setup-ap.service -n 100 --no-pager
```

Check portal service:

```bash
systemctl status portal.service --no-pager
journalctl -u portal.service -n 100 --no-pager
```

Check AutoDarts:

```bash
systemctl status autodarts.service --no-pager
journalctl -u autodarts.service -n 100 --no-pager
```

Check updater:

```bash
systemctl status autodarts-updater.timer --no-pager
systemctl list-timers autodarts-updater.timer
journalctl -u autodarts-updater.service -n 100 --no-pager
```

Check listening ports:

```bash
sudo ss -lntp
```

Expected services include:

- Board Manager HTTP on port `3180`
- Board Manager HTTPS on port `3181`
- SSH on port `22`
- Captive portal on port `80` only while setup mode is active

### Resetting WiFi setup

To force setup mode at a new location:

1. Disconnect Ethernet.
2. Ensure no saved WiFi network is available.
3. Reboot the Raspberry Pi.
4. Wait for `AutoDart-Setup`.

To inspect or remove saved NetworkManager profiles:

```bash
nmcli connection show
```

Remove a specific saved profile:

```bash
sudo nmcli connection delete "Connection name"
```

Then reboot:

```bash
sudo reboot
```

> ⚠️ **Warning:** Removing the active connection can immediately disconnect SSH. Perform this locally or be prepared to reconnect through `AutoDart-Setup`.

---

## Release Image Notes

The release image is intentionally distributed without:

- Personal WiFi credentials
- A claimed AutoDarts board identity
- Personal Board ID or API key information
- Temporary build files
- Backup files containing credentials

Each installation must connect to its own network and claim its own board.

Release files:

```text
autodarts-pi-v1.1.1.img.xz
```

---

*AutoDarts Pi Image • Technical Manual • v1.1.1 • By Johan Alvedal*
