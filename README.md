# 🎯 Autodartpi

> Headless Raspberry Pi image for AutoDarts – WiFi setup via captive portal, no monitor or keyboard required.

[![Release](https://img.shields.io/github/v/release/JohanAlvedal/Autodartpi)](https://github.com/JohanAlvedal/Autodartpi/releases/latest)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## ⬇️ Download

**[→ Download the latest AutoDarts Pi image](https://github.com/JohanAlvedal/Autodartpi/releases/latest)**

The release includes:

- `autodarts-pi-v1.1.1.img.xz`

Flash the `.img.xz` file directly with [Raspberry Pi Imager](https://www.raspberrypi.com/software/). No extraction is required.

---

## 🚀 Quick Start

| Step | Action |
|------|--------|
| 1 | Flash `autodarts-pi-v1.1.1.img.xz` to an SD card using Raspberry Pi Imager |
| 2 | Insert the SD card into the Raspberry Pi and power it on |
| 3 | Wait approximately 60–90 seconds for the first boot |
| 4 | Connect your phone or computer to the WiFi network **AutoDart-Setup** |
| 5 | The setup portal should open automatically – select your WiFi network and enter its password |
| 6 | The Raspberry Pi connects to your WiFi and the setup network disappears |
| 7 | Open the Board Manager at **http://autodartpi.local:3180** |
| 8 | Claim your board in the AutoDarts Board Manager |

> **Tip:** If the portal does not open automatically, navigate manually to `http://192.168.4.1`.

> **Note:** If Ethernet is connected, the setup access point is not started.

---

## 📋 Requirements

- Raspberry Pi 4 Model B with 2 GB RAM or more, or Raspberry Pi 5
- MicroSD card – minimum 8 GB, recommended 16 GB Class 10 / A1 or better
- USB-C power supply:
  - Raspberry Pi 4: 5V / 3A
  - Raspberry Pi 5: 5V / 5A recommended
- AutoDarts cameras and USB hub
- A phone or computer with WiFi for initial setup

---

## 🌐 Access

| Service | Address |
|---------|---------|
| Board Manager | http://autodartpi.local:3180 |
| Board Manager HTTPS | https://autodartpi.local:3181 |
| SSH | `ssh autodart@autodartpi.local` |
| Setup portal | http://192.168.4.1 |

The setup portal is only available while `AutoDart-Setup` is active.

---

## 🔐 Default SSH Credentials

| | |
|--|--|
| **User** | `autodart` |
| **Password** | `autodart` |

> ⚠️ Change the default password after your first SSH login:

```bash
passwd
````

---

## ⚙️ WiFi Pre-configuration

The captive portal can be skipped by creating or editing:

```text
/boot/firmware/autodarts.conf
```

Use this format:

```ini
WIFI_SSID="YourNetworkName"
WIFI_PASSWORD="YourPassword"
```

The WiFi network name is case-sensitive.

Board claiming is completed separately in the AutoDarts Board Manager.

---

## 📶 Network Behaviour

The Raspberry Pi checks available connections during startup:

* If Ethernet is connected, the setup access point is not started.
* If a saved WiFi network is available, the Raspberry Pi connects automatically.
* If no working network is found, `AutoDart-Setup` starts automatically.
* The setup access point remains available for up to 15 minutes.
* The captive portal stops automatically after successful WiFi configuration.

When moving the Raspberry Pi to a new location, start it without Ethernet. If no saved WiFi network is available, `AutoDart-Setup` will start again.

---

## 🔄 Automatic Updates

AutoDarts checks for updates automatically:

* five minutes after every boot
* every three days after that

To trigger an update manually:

```bash
ssh autodart@autodartpi.local
sudo systemctl start autodarts-updater.service
journalctl -u autodarts-updater.service -f
```

Check the next scheduled update with:

```bash
systemctl list-timers autodarts-updater.timer
```

---

## 🛠️ SSH & CLI Quick Reference

```bash
# Log in
ssh autodart@autodartpi.local

# Check AutoDarts status
systemctl status autodarts.service

# Restart AutoDarts
sudo systemctl restart autodarts.service

# View AutoDarts logs
journalctl -u autodarts.service -f

# Check the WiFi setup service
systemctl status setup-ap.service

# Check the update timer
systemctl status autodarts-updater.timer

# Reboot
sudo reboot
```

## 🔧 Troubleshooting

| Problem                            | Solution                                                                              |
| ---------------------------------- | ------------------------------------------------------------------------------------- |
| `AutoDart-Setup` is not visible    | Wait 60–90 seconds. Disconnect Ethernet and restart the Raspberry Pi                  |
| Portal does not open automatically | Navigate manually to `http://192.168.4.1`                                             |
| Wrong WiFi password                | Wait for the error message and try again                                              |
| Pi does not connect to WiFi        | Check the SSID, password, WiFi country setting and selected router channel            |
| Board Manager cannot be reached    | Try `http://autodartpi.local:3180` or find the Pi address in your router              |
| AutoDarts does not start           | Run `systemctl status autodarts.service` and `journalctl -u autodarts.service -n 100` |
| The board is not claimed           | Open Board Manager and follow the claim instructions                                  |
| Setup access point disappears      | This is expected after a successful WiFi connection                                   |
| Setup window expired               | Restart the Raspberry Pi without Ethernet to start setup mode again                   |

---

## 📖 Documentation

The full technical manual is available in:

[docs/MANUAL.md](docs/MANUAL.md)

---

## 📄 License

MIT License – see [LICENSE](LICENSE) for details.
