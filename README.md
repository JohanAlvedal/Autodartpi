# 🎯 Autodartpi

> Headless Raspberry Pi image for AutoDarts – WiFi setup via captive portal, no monitor or keyboard needed.

[![Release](https://img.shields.io/github/v/release/JohanAlvedal/Autodartpi)](https://github.com/JohanAlvedal/Autodartpi/releases/latest)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## ⬇️ Download

**[→ Download latest image (autodarts-pi-v1.1.img.xz)](https://github.com/JohanAlvedal/Autodartpi/releases/latest)**

Flash with [Raspberry Pi Imager](https://www.raspberrypi.com/software/) – supports `.img.xz` natively.

---

## 🚀 Quick Start

| Step | Action |
|------|--------|
| 1 | Flash `autodarts-pi-v1.1.img.xz` to SD card using Raspberry Pi Imager |
| 2 | Insert SD card into Raspberry Pi and power on |
| 3 | Wait ~30–60 seconds for the Pi to boot |
| 4 | Connect your phone or computer to the WiFi network **AutoDart-Setup** |
| 5 | A setup portal opens automatically – enter your WiFi credentials |
| 6 | The Pi connects to your WiFi and AutoDarts starts |
| 7 | Access the Board Manager at **http://autodartpi.local:3180** |

> **Tip:** If the portal doesn't open automatically, navigate manually to `http://192.168.4.1` in your browser.

---

## 📋 Requirements

- Raspberry Pi 4 Model B (2 GB RAM or more) or Raspberry Pi 5
- MicroSD card – minimum 8 GB, recommended 16 GB Class 10 / A1
- USB-C power supply (5V/3A for RPi 4, 5V/5A for RPi 5)
- AutoDarts camera and USB hub
- A phone or computer with WiFi for initial setup

---

## 🌐 Access

| What | Address |
|------|---------|
| Board Manager | http://autodartpi.local:3180 |
| SSH | `ssh autodart@autodartpi.local` |
| Setup portal (first boot) | http://192.168.4.1 |

---

## ⚙️ Pre-configuration (optional)

Skip the portal entirely by creating `/boot/firmware/autodarts.conf` on the SD card before first boot:

```ini
WIFI_SSID=YourNetworkName
WIFI_PASSWORD=YourPassword
AUTODARTS_BOARD_ID=your-board-id
AUTODARTS_API_KEY=your-api-key
```

---

## 🔄 Automatic Updates

AutoDarts updates itself automatically every week via a systemd timer. To trigger a manual update:

```bash
ssh autodart@autodartpi.local
sudo systemctl start autodarts-updater.service
journalctl -u autodarts-updater.service -f
```

---

## 🛠️ SSH & CLI Quick Reference

```bash
# Log in
ssh autodart@autodartpi.local

# Check status
systemctl status autodarts.service

# Restart AutoDarts
sudo systemctl restart autodarts.service

# View live logs
journalctl -u autodarts.service -f

# Reboot
sudo reboot
```

---

## 📁 Repository Structure

```
Autodartpi/
├── README.md
├── docs/
│   └── AutoDarts-Pi-Manual.docx
├── portal/
│   └── app.py                    # Captive portal (Flask)
├── scripts/
│   ├── setup-ap.sh               # WiFi AP logic
│   └── build-image.sh            # dd + pishrink + xz
├── systemd/
│   ├── setup-ap.service
│   ├── portal.service
│   ├── autodarts.service
│   └── autodarts-updater.service
└── config/
    └── autodarts.conf.example
```

---

## 🔧 Troubleshooting

| Problem | Solution |
|---------|----------|
| AutoDart-Setup not visible | Wait 60–90 seconds, then restart the Pi |
| Portal doesn't open automatically | Navigate manually to `http://192.168.4.1` |
| Wrong WiFi password | An error message appears after ~30 seconds – try again |
| Pi doesn't connect to WiFi | Check SSID (case-sensitive). RPi 4 does not support 5 GHz networks |
| AutoDarts doesn't start | Check Board ID and API key via SSH |
| AP shuts down too early | The AP runs for 15 minutes – restart Pi and connect quickly |

---

## 📖 Documentation

Full technical manual available in [`docs/AutoDarts-Pi-Manual.docx`](docs/AutoDarts-Pi-Manual.docx)

---

## 📄 License

MIT License – see [LICENSE](LICENSE) for details.
