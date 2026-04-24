# AutoDarts Pi Image
## Technical Manual & Deployment Guide

> Headless Raspberry Pi setup with WiFi captive portal and automatic updates  
> Version 1.1 • Raspberry Pi OS Lite 64-bit  
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
7. [Automatic Updates](#7-automatic-updates)
8. [SSH Access & CLI Management](#8-ssh-access--cli-management)
9. [System Files & Configuration](#9-system-files--configuration)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. System Overview

AutoDarts Pi Image is a pre-built Raspberry Pi installation that allows you to deploy AutoDarts hardware without a monitor, keyboard, or network cable. Everything is configured wirelessly via a built-in captive portal.

| Property | Value |
|----------|-------|
| Base OS | Raspberry Pi OS Lite 64-bit |
| Target Hardware | Raspberry Pi 4 / Raspberry Pi 5 |
| SD Card (min) | 8 GB Class 10 / A1 |
| Hostname | `autodartpi.local` |
| WiFi AP SSID | `AutoDart-Setup` |
| Portal Address | http://192.168.4.1 |
| Board Manager | http://autodartpi.local:3180 |
| SSH Access | `ssh autodart@autodartpi.local` |
| Auto-update | On every boot (5 min delay), then every 3 days |

### Process Overview

| Step | Action |
|------|--------|
| 1 | Flash the `.img.xz` file to an SD card using Raspberry Pi Imager |
| 2 | *(Optional)* Pre-configure WiFi in `/boot/firmware/autodarts.conf` |
| 3 | Boot the Pi – the system boots headless in 30–90 seconds |
| 4 | Connect your phone/computer to the WiFi network **AutoDart-Setup** |
| 5 | Enter your WiFi credentials in the portal at `192.168.4.1` |
| 6 | The Pi connects to your WiFi and starts AutoDarts automatically |
| 7 | Access the Board Manager at **http://autodartpi.local:3180** |
| 8 | AutoDarts updates automatically on every boot and every 3 days |

---

## 2. Requirements & Hardware

### Required

- Raspberry Pi 4 Model B (2 GB RAM or more) or Raspberry Pi 5
- MicroSD card, minimum 8 GB, recommended 16 GB Class 10 or A1/A2
- USB-C power supply (5V/3A for RPi 4, 5V/5A for RPi 5)
- AutoDarts camera and USB hub (per AutoDarts specifications)
- A computer with an SD card reader to flash the image

### Recommended

- Raspberry Pi Imager installed on your computer (free, from [raspberrypi.com/software](https://www.raspberrypi.com/software/))
- A phone or computer with WiFi for portal configuration

> **Note:** The Pi does NOT need to be connected to a monitor or keyboard. The entire setup is headless.

---

## 3. Flashing the Image to SD Card

The image is distributed as a compressed `.img.xz` file. Raspberry Pi Imager can flash it directly without extracting.

### 3.1 Using Raspberry Pi Imager (recommended)

Raspberry Pi Imager is the easiest method and works on Windows, macOS and Linux.

| Step | Action |
|------|--------|
| 1 | Download and install Raspberry Pi Imager from [raspberrypi.com/software](https://www.raspberrypi.com/software/) |
| 2 | Insert the SD card into your computer |
| 3 | Launch Imager and click **Choose OS** |
| 4 | Scroll down and select **Use custom** at the bottom of the list |
| 5 | Select the file `autodarts-pi.img.xz` (Imager handles `.xz` natively) |
| 6 | Click **Choose Storage** and select your SD card *(be careful – wrong disk will be erased)* |
| 7 | Click **Write** and confirm |
| 8 | Wait until the process completes (~5–15 minutes depending on SD card speed) |
| 9 | Safely eject the SD card and insert it into the Pi |

> **Tip:** Raspberry Pi Imager verifies the flash automatically. If verification fails, try a different SD card.

### 3.2 Using balenaEtcher

balenaEtcher is an alternative tool that also works on Windows, macOS and Linux.

| Step | Action |
|------|--------|
| 1 | Download balenaEtcher from [etcher.balena.io](https://etcher.balena.io/) |
| 2 | Open Etcher and click **Flash from file** |
| 3 | Select `autodarts-pi.img.xz` |
| 4 | Select your SD card under **Select target** |
| 5 | Click **Flash!** and wait for completion |

### 3.3 Using dd (Linux/Mac – advanced)

For advanced users who prefer the command line. Be extra careful to specify the correct disk – choosing the wrong one will permanently erase data.

Decompress and flash in a single command:

```bash
xzcat autodarts-pi.img.xz | sudo dd of=/dev/sdX bs=4M status=progress
```

Replace `/dev/sdX` with your SD card device. Verify first:

```bash
# Linux
lsblk

# Mac
diskutil list
```

> ⚠️ **Warning:** The `dd` command gives no confirmation dialog. Always double-check that `of=` points to the SD card and not your hard drive.

---

## 4. Pre-configuration via autodarts.conf

If you already know your WiFi password, you can skip the portal entirely. Create or edit the file `/boot/firmware/autodarts.conf` on the SD card before inserting it into the Pi.

The file is read by the system on first boot:

```ini
# /boot/firmware/autodarts.conf
WIFI_SSID=YourNetworkName
WIFI_PASSWORD=YourPassword
AUTODARTS_BOARD_ID=your-board-id
AUTODARTS_API_KEY=your-api-key
```

| Parameter | Description |
|-----------|-------------|
| `WIFI_SSID` | Name of your WiFi network (case-sensitive) |
| `WIFI_PASSWORD` | WiFi password in plain text |
| `AUTODARTS_BOARD_ID` | Your AutoDarts Board ID from autodarts.io |
| `AUTODARTS_API_KEY` | Your API key from autodarts.io |

> **Note:** If the file is missing or empty, the system will start the WiFi portal automatically.

---

## 5. First Boot – Step by Step

### 5.1 Boot sequence in detail

| Time | Event |
|------|-------|
| 0:00 | Power connected – Pi begins booting |
| 0:05 | Bootloader reads SD card, kernel is loaded |
| 0:15 | Linux starts, filesystem is mounted and resized (first boot only) |
| 0:25 | systemd starts all services in parallel |
| 0:30 | `setup-ap.service` activates – checks `autodarts.conf` |
| 0:32 | If conf is missing: WiFi AP **AutoDart-Setup** is started (`192.168.4.1`) |
| 0:33 | `portal.service` starts the Flask application on port 80 |
| 0:90 | If conf exists: WiFi connects directly, AutoDarts starts |

### 5.2 What happens in the background?

#### setup-ap.service

This service is the heart of the entire setup flow. The script `/usr/local/bin/setup-ap.sh` runs and does the following:

- Checks whether `/boot/firmware/autodarts.conf` exists and is populated
- If pre-configuration is found: Configures WiFi using `nmcli` and proceeds to AutoDarts start
- If no configuration exists: Creates a WiFi access point using NetworkManager
- The AP runs SSID `AutoDart-Setup` with no password on `192.168.4.1/24`
- A built-in timer starts – if nobody connects within 15 minutes, the AP shuts down automatically

#### portal.service

The Flask web application at `/home/autodart/portal/app.py` starts on port 80. It presents a two-step wizard: first WiFi credentials, then AutoDarts Board ID and API key. The portal handles:

- WiFi SSID selection (scanned live from available networks) and password input
- A visual spinner and status message shown immediately while connecting
- A clear error message if the password is incorrect (after ~30 seconds)
- Optional AutoDarts Board ID and API key input
- Saving configuration to `/boot/firmware/autodarts.conf`
- Calling `nmcli` to connect the Pi to the specified network
- Shutting down the AP and restarting the AutoDarts service

> **Note:** If an incorrect WiFi password is entered, a loading indicator appears immediately and a clear error is shown after ~30 seconds. This delay is a known limitation of how NetworkManager handles authentication failures.

#### autodarts.service

The AutoDarts software is installed under `/home/autodart` and runs as a systemd service. On start:

- The program reads its configuration from `/home/autodart/.config/autodarts/config.toml`
- It connects to the AutoDarts cloud service using the API key and Board ID
- Camera input is initialized and dart detection begins
- The Board Manager is accessible at **http://autodartpi.local:3180**

#### autodarts-updater.service + timer

A systemd timer triggers the updater in two situations: 5 minutes after every boot, and then on a recurring 3-day interval. The update:

- Runs automatically without any user interaction
- Stops AutoDarts, downloads the latest version, then restarts it
- Is logged to the systemd journal (`journalctl -u autodarts-updater`)

---

## 6. WiFi Setup via Captive Portal

### 6.1 Connecting to AutoDart-Setup

About 30 seconds after the Pi boots, a new open WiFi network becomes visible:

| Property | Detail |
|----------|--------|
| SSID | `AutoDart-Setup` |
| Password | *(none – open network)* |
| Pi IP address | `192.168.4.1` |
| Your IP (DHCP) | `192.168.4.x` |

Connect your phone or computer to `AutoDart-Setup`. Most devices will open a captive portal dialog automatically. If not, open a browser and navigate manually to:

```
http://192.168.4.1
```

> **Tip:** Disable mobile data on your phone if the portal doesn't appear – otherwise the phone may route traffic through the cellular network instead.

### 6.2 Filling in the portal

The portal presents a two-step wizard:

**Step 1 – WiFi:** Select your network from the dropdown and enter your password. A spinner and status message appear immediately while the Pi connects. If the password is wrong, a clear error message is shown after ~30 seconds so you can try again.

**Step 2 – AutoDarts:** Enter your Board ID and API Key from [autodarts.io](https://autodarts.io) (or skip for now).

### 6.3 What happens after saving?

| Step | Action |
|------|--------|
| 1 | Form data is sent to the Flask application via HTTP POST |
| 2 | `app.py` saves the credentials to `/boot/firmware/autodarts.conf` |
| 3 | `nmcli` is called to configure and connect to your WiFi network |
| 4 | The AutoDart-Setup WiFi AP is shut down |
| 5 | `portal.service` exits |
| 6 | AutoDarts starts and connects to the cloud service |
| 7 | The Pi is now ready – access the Board Manager at **http://autodartpi.local:3180** |

> **Note:** After step 4 you will lose your connection to AutoDart-Setup – this is normal and expected. The Pi is now connected to your WiFi.

---

## 7. Automatic Updates

The system updates AutoDarts automatically on every boot (after a 5-minute delay) and then every 3 days. No manual action is required.

### Checking update status

```bash
# View the updater service log
journalctl -u autodarts-updater.service -n 50

# See when the next update is scheduled
systemctl list-timers autodarts-updater.timer
```

### Manual update

```bash
sudo systemctl start autodarts-updater.service

# Follow the update live
journalctl -u autodarts-updater.service -f
```

> **Note:** AutoDarts will restart automatically after an update. Any game in progress will be interrupted.

---

## 8. SSH Access & CLI Management

Once the Pi is on your network, connect via SSH to manage services and check logs.

### Logging in

```bash
ssh autodart@autodartpi.local
```

> **Tip:** The hostname `autodartpi.local` resolves via mDNS. Works natively on Mac and Linux. Windows users may need to install Bonjour or use the Pi's IP address directly.

### Quick reference

| Task | Command |
|------|---------|
| Log in via SSH | `ssh autodart@autodartpi.local` |
| Open Board Manager | http://autodartpi.local:3180 |
| Check AutoDarts status | `systemctl status autodarts.service` |
| Restart AutoDarts | `sudo systemctl restart autodarts.service` |
| Trigger manual update | `sudo systemctl start autodarts-updater.service` |
| Follow logs live | `journalctl -u autodarts.service -f` |
| View config file | `cat /boot/firmware/autodarts.conf` |
| Edit config file | `sudo nano /boot/firmware/autodarts.conf` |
| Reboot Pi | `sudo reboot` |

### Checking service status

```bash
systemctl status autodarts.service
systemctl status autodarts-updater.service
systemctl status portal.service
systemctl status setup-ap.service
```

### Useful logs

```bash
# All logs since last boot
journalctl -b

# AutoDarts logs today
journalctl -u autodarts.service --since today

# Last 50 updater lines
journalctl -u autodarts-updater.service -n 50

# Live log stream
journalctl -f
```

---

## 9. System Files & Configuration

| File / Path | Description |
|-------------|-------------|
| `/usr/local/bin/setup-ap.sh` | Main script for WiFi AP management and the setup flow |
| `/home/autodart/portal/app.py` | Flask captive portal – the web interface |
| `/etc/systemd/system/setup-ap.service` | systemd service for AP management |
| `/etc/systemd/system/portal.service` | systemd service for the Flask portal |
| `/etc/systemd/system/autodarts-updater.service` | systemd service for AutoDarts updates |
| `/boot/firmware/autodarts.conf` | Pre-configuration file (WiFi, Board ID, API key) |
| `/home/autodart/.config/autodarts/config.toml` | AutoDarts runtime configuration |

---

## 10. Troubleshooting

| Problem | Possible solution |
|---------|-------------------|
| AutoDart-Setup not visible | Wait 60–90 seconds. Check that the Pi has power. Restart the Pi. |
| Portal doesn't open automatically | Navigate manually to `http://192.168.4.1` in a browser |
| Portal shows error after WiFi input | Check that the password is correct. The error appears after ~30 sec. |
| Pi doesn't connect to WiFi | Verify SSID (case-sensitive) and password. 5 GHz networks are not supported on RPi 4. |
| AutoDarts doesn't start | Check Board ID and API key in `autodarts.conf` |
| AP shuts down too early | The AP lives for 15 minutes. Connect quickly after boot, or restart the Pi. |
| Can't reach autodartpi.local | Try the Pi's IP address directly. Check that mDNS is supported on your device. |
| Board Manager not loading | Check that `autodarts.service` is running: `systemctl status autodarts.service` |

### Useful logs to check

```bash
# System logs since last boot
journalctl -b

# Specific services
journalctl -u setup-ap.service --since today
journalctl -u portal.service --since today
journalctl -u autodarts.service --since today
journalctl -u autodarts-updater.service --since today
```

---

*AutoDarts Pi Image • Technical Manual • v1.1 • By Johan Alvedal*
