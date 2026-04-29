<p align="right">
  <img src="assets/super.png" width="48" alt="Super">
</p>

# Raspberry-Pi-Headless-Setup

How to set up a Raspberry Pi with every step explained. Assumes you want to run "headless" (without a monitor and keyboard).

> **Heads up:** Since Raspberry Pi OS Bookworm (Dec 2023) the legacy `ssh` / `wpa_supplicant.conf` boot-partition trick has been deprecated, and there is no longer a default `pi` / `raspberry` user. The recommended path below uses the official Raspberry Pi Imager to pre-configure SSH, Wi-Fi, username/password, and locale before first boot.

## Contents

- [Prerequisites](#prerequisites)
- [Step-by-step](#step-by-step)
  - [Step 1: Install the Raspberry Pi Imager](#step-1-install-the-raspberry-pi-imager)
  - [Step 2: Choose the OS and pre-configure the image](#step-2-choose-the-os-and-pre-configure-the-image)
  - [Step 3 (fallback): Manual configuration for older OS releases](#step-3-fallback-manual-configuration-for-older-os-releases)
  - [Step 4: Power up your Raspberry Pi](#step-4-power-up-your-raspberry-pi)
  - [Step 5: Find the Pi on your network](#step-5-find-the-pi-on-your-network)
  - [Step 6: SSH into the Pi](#step-6-ssh-into-the-pi)
- [Optional](#optional)
  - [Step 7: Wi-Fi via NetworkManager](#step-7-wi-fi-via-networkmanager)
  - [Step 8: Set up the Raspberry Pi OS GUI via VNC](#step-8-set-up-the-raspberry-pi-os-gui-via-vnc)
  - [Step 9: Connect to the Pi GUI](#step-9-connect-to-the-pi-gui)
- [Troubleshooting](#troubleshooting)

## Prerequisites

- A Raspberry Pi (any model with Wi-Fi if you plan to skip Ethernet) and a compatible power supply
- An SD card (8 GB or larger, Class 10 recommended) and a way to write it from your PC/Mac
- Your Wi-Fi SSID and password
- Your PC/Mac on the same network as the Pi will join

## Step-by-step

### Step 1: Install the Raspberry Pi Imager

Download and install the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) on your PC or Mac.

Insert the SD card you want to write to.

### Step 2: Choose the OS and pre-configure the image

1. Click `CHOOSE DEVICE` and select your Pi model.
2. Click `CHOOSE OS` and pick **Raspberry Pi OS** (the default, 64-bit where available).
3. Click `CHOOSE STORAGE` and select your SD card.
4. Click `NEXT`, then when prompted to apply OS customisation settings choose **Edit Settings** (or press `Ctrl`+`Shift`+`X` in older Imager versions).

In the customisation dialog, set:

- **Hostname** (e.g. `raspberrypi`) — this is how you'll reach the Pi on your LAN.
- **Username and password** — required; there is no longer a default user.
- **Wireless LAN** — SSID, password, and Wi-Fi country (uppercase ISO 3166-1, e.g. `GB`, `US`, `DE`).
- **Locale** — time zone and keyboard layout.

On the **Services** tab, enable **SSH** and choose either password or public-key authentication (public-key is recommended).

Click `SAVE`, then `YES` to apply settings and write the image. Eject the SD card when it finishes.

> **Security note:** Choose a strong password and prefer public-key SSH authentication. Avoid reusing credentials; a Pi exposed to the internet without hardening is a common attack target.

### Step 3 (fallback): Manual configuration for older OS releases

Skip this step if you used the Imager customisation in Step 2.

If you must image a pre-Bookworm release, you can enable SSH and Wi-Fi manually on the **boot** partition of the SD card (the small FAT partition visible on Windows/macOS, not the Linux rootfs):

1. Create an empty file named `ssh` (no extension) in the boot partition to enable SSH.
2. Create a file named `wpa_supplicant.conf` in the same boot partition with the following content:

   ```conf
   country=GB
   update_config=1
   ctrl_interface=/var/run/wpa_supplicant
   network={
       scan_ssid=1
       ssid="MyNetworkSSID"
       psk="MyPassword"
   }
   ```

   Replace `country` with your uppercase ISO 3166-1 code and the SSID/PSK with your network details. To avoid storing the plaintext password you can generate a hashed PSK with `wpa_passphrase "MyNetworkSSID" "MyPassword"` on any Linux machine.

Safely eject the SD card.

### Step 4: Power up your Raspberry Pi

Insert the SD card into the Pi and connect power. Give it a minute or two to boot and connect to Wi-Fi.

### Step 5: Find the Pi on your network

You can connect by hostname if mDNS is available:

```bash
ping raspberrypi.local
```

If `.local` resolution doesn't work (common on some Windows setups), find the IP via:

- your router's admin page / DHCP client list, or
- a LAN scanner such as `nmap -sn 192.168.1.0/24` or the mobile app **Fing**.

### Step 6: SSH into the Pi

From macOS/Linux Terminal or Windows PowerShell:

```bash
ssh <your-username>@raspberrypi.local
```

Use the username and password you set in Step 2. Accept the host-key prompt on first connection.

## Optional

### Step 7: Wi-Fi via NetworkManager

Bookworm switched to NetworkManager by default. See [WiFi Configuration via NetworkManager on RPi](https://github.com/sraodev/Raspberry-Pi-Headless-Setup-via-Network-Manager) for CLI (`nmcli`) examples.

### Step 8: Set up the Raspberry Pi OS GUI via VNC

Update packages and install RealVNC:

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt install -y realvnc-vnc-server realvnc-vnc-viewer
```

Enable the VNC service:

```bash
sudo raspi-config
```

Navigate to **Interface Options > VNC > Yes**, exit, and reboot:

```bash
sudo reboot
```

### Step 9: Connect to the Pi GUI

Download and open [VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/). Enter the Pi's IP address or hostname and connect using the credentials you set in Step 2.

On first GUI login you may be prompted to:

- confirm locale and keyboard layout,
- change your password (recommended if you haven't already),
- skip re-entering Wi-Fi details (they're already configured),
- install pending updates.

After setup, adjust the screen resolution via **Raspberry menu > Preferences > Raspberry Pi Configuration > Display > Set Resolution**, then reboot for the change to take effect.

## Troubleshooting

**`raspberrypi.local` does not resolve.**
mDNS isn't always available — notably on older Windows hosts without Bonjour. Install [Bonjour Print Services](https://support.apple.com/kb/dl999) on Windows, or skip the hostname and use the Pi's IP directly. Find it via your router's DHCP client list, `nmap -sn 192.168.1.0/24`, or the **Fing** mobile app. If you set a custom hostname in the Imager, substitute it for `raspberrypi`.

**Pi never joins Wi-Fi.**
Most often caused by a missing or wrong country code, an SSID typo, or 5 GHz-only Wi-Fi on a Pi model that's 2.4 GHz-only (Pi Zero W, Pi 3A+ in 5 GHz dead zones). Re-image with the Imager and double-check the country (uppercase ISO 3166-1, e.g. `GB`, `US`, `DE`). For the manual fallback, confirm `wpa_supplicant.conf` lives on the **boot** partition (FAT) and not the Linux rootfs.

**SSH refuses to connect / "Connection refused".**
The Pi may not have finished booting — wait 60–90 seconds after power-up. If using the manual fallback, verify the empty `ssh` file is on the boot partition with no extension (Windows often hides `.txt`).

**SSH host-key warning after re-imaging.**
Re-flashing the SD card regenerates the Pi's host key, so your client refuses the new fingerprint. Remove the stale entry:

```bash
ssh-keygen -R raspberrypi.local
ssh-keygen -R <pi-ip-address>
```

**"Permission denied (publickey,password)".**
The username does not exist (you're probably trying the legacy `pi` user on a Bookworm-or-newer image). Use the username you set in Step 2.

**`apt` fails with hash-sum mismatch or 404.**
Usually transient mirror issues. Retry, or run `sudo apt clean && sudo apt update`.

**VNC Viewer shows "cannot currently show the desktop".**
Enable a virtual framebuffer in `sudo raspi-config` → **Display Options > VNC Resolution**, set a resolution (e.g. 1280×720), then reboot.

---

[Reference](https://www.hackster.io/mark-hank/super-simple-headless-raspberry-pi-setup-5382d6)
