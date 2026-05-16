# 🎯 AutoDarts Pi – Headless Raspberry Pi Image

A ready-to-flash Raspberry Pi image for AutoDarts with zero-configuration setup via a mobile captive portal. No keyboard, mouse, or monitor needed.

---

## What's Included

- Raspberry Pi OS Lite 64-bit
- AutoDarts (latest version via get.autodarts.io)
- WiFi setup portal (captive portal on 192.168.4.1)
- Auto-start on boot via systemd
- Pre-configuration via `autodarts.conf`
- Automatic weekly AutoDarts updates

---

## How It Works

```
Boot → Check autodarts.conf
     → WiFi credentials found? → Connect automatically
     → No credentials? → Start "AutoDart-Setup" WiFi AP
          → Connect phone → Open http://192.168.4.1
          → Select WiFi + password → Enter Board ID + API Key
          → Done! AutoDarts starts automatically
     → No one connects within 15 min → AP shuts down
```

---

## Quick Start

### 1. Flash the Image

Download and install [Raspberry Pi Imager](https://www.raspberrypi.com/software/)

| Step | Action |
|------|--------|
| 1 | Open Raspberry Pi Imager |
| 2 | Choose Device → Raspberry Pi 4 |
| 3 | Choose OS → Use custom → select `autodart.img.xz` |
| 4 | Choose Storage → your SD card |
| 5 | Click Next → **No** on customisation (already built-in) → Yes |
| 6 | Wait for flash to complete |

### 2a. Option A – Pre-configure via file (easiest)

After flashing, open the SD card in Windows Explorer. You will see a file called `autodarts.conf`. Edit it:

```ini
# Optional: WiFi credentials
WIFI_SSID=YourNetworkName
WIFI_PASSWORD=YourPassword

# Optional: AutoDarts credentials
BOARD_ID=your-board-id
API_KEY=your-api-key
```

Leave any field empty to configure it later via the portal.

### 2b. Option B – Configure via mobile portal

1. Insert SD card into Raspberry Pi and power on
2. Wait ~30 seconds
3. Connect your phone to WiFi network **"AutoDart-Setup"** (no password)
4. Open browser and go to **http://192.168.4.1**
5. Select your home WiFi and enter password
6. Optionally enter your Board ID and API Key
7. Click Connect – done!

---

## Requirements

| Item | Spec |
|------|------|
| Raspberry Pi | 4 Model B (also works on RPi 5) |
| SD Card | 8 GB minimum, 16 GB+ recommended |
| Power Supply | Official RPi power supply recommended |
| Cameras | USB cameras per AutoDarts specifications |
| Phone/Laptop | For initial WiFi setup |

> **Note:** Active cooling is recommended for RPi 4. Use 2.4 GHz WiFi for best stability.

---

## After Setup

Access the AutoDarts Board Manager from any device on your network:

```
http://autodartpi.local:3180
```

If you haven't registered your board yet:
1. Go to [autodarts.io](https://autodarts.io)
2. Register a new board
3. Copy your Board ID and API Key
4. Enter them in the Board Manager

---

## System Files

| File | Description |
|------|-------------|
| `/usr/local/bin/setup-ap.sh` | WiFi AP logic and boot flow |
| `/home/autodart/portal/app.py` | Flask captive portal |
| `/etc/systemd/system/setup-ap.service` | AP systemd service |
| `/etc/systemd/system/portal.service` | Portal systemd service |
| `/etc/systemd/system/autodarts-updater.timer` | Weekly update timer |
| `/boot/firmware/autodarts.conf` | Pre-configuration file |
| `/home/autodart/.config/autodarts/config.toml` | AutoDarts runtime config |
| `/home/autodart/build-image.sh` | Script to build a new image |

---

## Checking Service Status (via SSH)

```bash
ssh autodart@autodartpi.local

# Check all services
sudo systemctl status setup-ap portal autodarts

# Follow live logs
journalctl -f

# Check update timer
systemctl list-timers autodarts-updater.timer
```

---

## Manual AutoDarts Update

```bash
curl -sSL https://get.autodarts.io | bash
```

---

## Building a New Image

If you modify the system and want to create a new distributable image:

1. Connect a USB drive (32 GB+) to the Raspberry Pi
2. Run the build script:

```bash
sudo /home/autodart/build-image.sh
```

3. Transfer to Windows:

```powershell
scp autodart@autodartpi.local:/mnt/usb/autodart.img.xz C:\Users\YourUser\Desktop\autodart.img.xz
```

4. Flash with Raspberry Pi Imager as usual

> **Before building:** Make sure to clear your credentials from `autodarts.conf` and `config.toml` so the image is clean for other users.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| AutoDart-Setup not visible | Wait 60 seconds, restart Pi |
| Portal doesn't open automatically | Navigate manually to http://192.168.4.1 |
| Wrong WiFi password | Portal will show an error, try again |
| Pi doesn't connect to WiFi | Check SSID (case-sensitive), 5 GHz not supported on RPi 4 |
| AutoDarts doesn't start | Check Board ID and API Key in Board Manager |
| AP shuts down too early | AP stays up for 15 minutes, restart Pi to try again |

---

## Technical Details

### Boot Flow

```
0:00  Power on
0:05  Bootloader, kernel loaded
0:15  Linux starts, filesystem mounted
0:25  systemd starts services
0:30  setup-ap.service runs
0:32  Reads /boot/firmware/autodarts.conf
0:35  If WiFi configured → connect, skip AP
0:35  If no WiFi → start AutoDart-Setup AP
0:40  portal.service starts Flask on port 80
0:45  autodarts.service starts
```

### AP Timeout Logic

- AP starts if no saved WiFi is found
- AP stays active for **15 minutes**
- If user configures WiFi during this window → AP shuts down, AutoDarts starts
- If nobody connects → AP shuts down automatically

---

## Compatibility

| Hardware | Supported |
|----------|-----------|
| Raspberry Pi 4 | ✅ Recommended |
| Raspberry Pi 5 | ✅ Works (requires active cooling + official PSU) |
| Raspberry Pi 3 | ⚠️ Not tested |

---

*AutoDarts Pi Image – Community project. Not affiliated with autodarts.io.*
