# macOS WiFi Security Guide
### Audit your network, identify unknown devices, and lock things down

---

## Table of Contents

1. [Before You Start](#1-before-you-start)
2. [Understanding the Basics](#2-understanding-the-basics)
3. [Check Your Router's Device List](#3-check-your-routers-device-list)
4. [Install nmap on macOS](#4-install-nmap-on-macos)
5. [Find Your Network Range](#5-find-your-network-range)
6. [Scan Your Network with nmap](#6-scan-your-network-with-nmap)
7. [Reading the Scan Results](#7-reading-the-scan-results)
8. [Investigating Unknown Devices](#8-investigating-unknown-devices)
9. [Harden Your Router](#9-harden-your-router)
10. [Additional macOS Security Tips](#10-additional-macos-security-tips)
11. [Quick Reference Cheat Sheet](#11-quick-reference-cheat-sheet)

---

## 1. Before You Start

This guide walks you through auditing your WiFi network from your Mac. You will use Terminal and a free tool called `nmap`. You do not need to be a security expert, but you do need to be comfortable running commands in Terminal.

**What you need:**
- A Mac connected to the WiFi network you want to audit
- Access to your router's admin page (the login page, usually at `192.168.1.1` or `192.168.0.1`)
- About 20-30 minutes

**Important:** Only scan networks you own or have explicit permission to scan. This guide is for your home or office network.

---

## 2. Understanding the Basics

Before scanning anything, it helps to know what you're looking for.

### IP Address
Every device on your network gets a local IP address, something like `192.168.1.105`. Think of it as a temporary seat number assigned by your router. It can change when a device reconnects.

### MAC Address
A MAC address is a hardware identifier built into a device's network card, like `BA:BD:59:D7:B5:27`. Unlike IP addresses, it's tied to the physical hardware. The first 6 characters (the first 3 pairs) identify the manufacturer.

**Note on MAC Randomization:** Modern phones (iPhone since iOS 14, most Android phones too) randomize their MAC address per network as a privacy feature. This means a phone might show an "Unknown" vendor even if it's a perfectly normal device you own.

### What "scanning" means
When you run a network scan, you're essentially sending a message to every possible address on your network and asking "is anyone there?" Devices that respond get listed. It's non-destructive and passive, you're just listening for who's home.

---

## 3. Check Your Router's Device List

Before using any tools, your router already has a list of connected devices. This is the quickest first step.

**How to access your router:**

1. Open any browser and go to one of these addresses:
   - `192.168.1.1`
   - `192.168.0.1`
   - `192.168.254.254` (common with Globe in the Philippines)
   - Or check the sticker on your router for the gateway address

2. Log in. If you've never changed the credentials, they're usually on the sticker on the router itself. Common defaults are `admin / admin` or `admin / password`.

3. Look for a section called **Connected Devices**, **DHCP Client List**, **Device Manager**, or **Wireless Clients**. The exact name depends on your router brand.

4. You'll see a table with device names, IP addresses, and MAC addresses.

**What to do with this list:**
Write down or screenshot the list. You'll compare it against your nmap results shortly.

---

## 4. Install nmap on macOS

`nmap` (Network Mapper) is a free, open-source tool used by security professionals worldwide. You'll install it via Homebrew, which is a package manager for Mac.

### Step 1: Install Homebrew (if you don't have it)

Open Terminal (press `Cmd + Space`, type "Terminal", hit Enter) and run:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Follow the on-screen prompts. It will ask for your Mac password.

**To verify Homebrew installed correctly:**
```bash
brew --version
```

You should see a version number like `Homebrew 4.x.x`.

### Step 2: Install nmap

```bash
brew install nmap
```

**To verify nmap installed correctly:**
```bash
nmap --version
```

You should see output starting with `Nmap version 7.x.x`.

---

## 5. Find Your Network Range

Before scanning, you need to know what range of addresses to scan.

Run this in Terminal:

```bash
ifconfig en0 | grep inet
```

You'll see output like this:

```
inet6 fe80::1c2c:4966:1663:10d2%en0 prefixlen 64 secured scopeid 0xb
inet 192.168.254.102 netmask 0xffffff00 broadcast 192.168.254.255
inet6 2001:fd8:... prefixlen 64 autoconf secured
```

Find the line that starts with `inet` (not `inet6`). That's your local IPv4 address.

In the example above: `192.168.254.102`

**Your network range** is that address with the last number replaced by `0`, followed by `/24`:

```
192.168.254.0/24
```

The `/24` tells nmap to scan all 254 possible addresses in that range (from `.1` to `.254`).

> **Note:** If you're on WiFi but `en0` shows nothing useful, try `en1` instead: `ifconfig en1 | grep inet`

---

## 6. Scan Your Network with nmap

Now you're ready to scan. There are a few different scans, starting from simple to more detailed.

### Basic Ping Scan (recommended starting point)

This just finds which devices are alive on the network. Fast and non-intrusive.

```bash
sudo nmap -sn 192.168.254.0/24
```

Replace `192.168.254.0/24` with your actual network range from Step 5.

You need `sudo` because without it, nmap can't retrieve MAC addresses on macOS. It will ask for your Mac login password.

**Example output:**
```
Starting Nmap 7.99 at 2026-04-17 15:47
Nmap scan report for 192.168.254.101
Host is up (0.0057s latency).
MAC Address: BA:BD:59:D7:B5:27 (Unknown)

Nmap scan report for globebroadband.net (192.168.254.254)
Host is up (0.0040s latency).
MAC Address: 58:AE:F1:C4:DA:F8 (Fiberhome Telecommunication Technologies)

Nmap scan report for 192.168.254.102
Host is up.

Nmap done: 256 IP addresses (3 hosts up) scanned in 5.12 seconds
```

### Service Detection Scan (dig deeper on a specific device)

Once you have an IP from the ping scan, you can dig into a specific device:

```bash
sudo nmap -sV 192.168.254.101
```

This tries to figure out what services and ports that device is running.

### Force Scan (if a device blocks ping but you know it's there)

Some devices block ping probes but are still active. Use `-Pn` to skip the ping check:

```bash
sudo nmap -Pn -sV 192.168.254.101
```

---

## 7. Reading the Scan Results

Here's how to interpret what nmap gives you.

### Each device block shows:
- **IP address** - its current address on your network
- **Hostname** - a friendly name, if available (like `globebroadband.net`)
- **Latency** - how fast it responded, not very important for this purpose
- **MAC Address** - the hardware ID, and in parentheses, the manufacturer nmap could identify

### MAC Address Vendors

The vendor name next to the MAC address is one of your best clues. If nmap says "Unknown," it doesn't necessarily mean suspicious. It often means:
- The device uses MAC address randomization (most modern phones do)
- It's a newer device whose MAC prefix isn't in nmap's database yet

You can look up any MAC address manually at:
- [maclookup.app](https://maclookup.app)
- [macvendors.com](https://macvendors.com)

Just enter the first 6 characters (first 3 pairs) of the MAC address.

### TTL Values as a Clue

When you run `ping` against a device, the TTL (Time to Live) value in the response gives a rough hint about the OS:

| TTL Value | Likely OS |
|-----------|-----------|
| 64 | Linux, Android, macOS, iOS |
| 128 | Windows |
| 255 | Network equipment (routers, switches) |

This is not definitive, but it helps narrow things down.

### Your Own Device

Your Mac will always appear in the list but without a MAC address shown. That's normal. nmap doesn't report your own device's MAC.

---

## 8. Investigating Unknown Devices

Found something you don't recognize? Here's a systematic way to identify it.

### Step 1: Count your devices

Make a list of everything you know is connected to your WiFi:
- Phones (yours and anyone else in the household)
- Laptops and tablets
- Smart TV or streaming sticks (Chromecast, Apple TV, Fire Stick)
- Game consoles
- Smart speakers (Google Home, Amazon Echo, HomePod)
- Smart home devices (thermostats, cameras, bulbs)
- Printers
- Network-attached storage (NAS)

It adds up fast. Most people have 8-15 devices without realizing it.

### Step 2: Check each device's IP

On each device, go to its WiFi settings and look for the assigned IP address. You're trying to match it to the unknown IP from your scan.

- **iPhone/iPad:** Settings > WiFi > tap the network name > shows IP Address
- **Android:** Settings > WiFi > tap the network > Advanced
- **Smart TV:** Network settings vary by brand, usually under Settings > Network > Status

### Step 3: The process of elimination

If you can't find it through settings, try this:

Turn off WiFi on each device one at a time, then re-run the ping scan after each one:

```bash
sudo nmap -sn 192.168.254.0/24
```

When the unknown IP disappears from the results, the device you just turned off is the one.

### Step 4: If you genuinely cannot account for it

If after all of the above you still can't identify the device, treat it as suspicious.

**Immediate steps:**
1. Change your WiFi password. This kicks every device off the network. Only your known devices should reconnect.
2. Check if your router supports MAC address filtering, you can whitelist only your devices.
3. Check your router's logs if it has them. Look for unusual connection times or high data usage from that IP.

---

## 9. Harden Your Router

The router is the front door of your network. These settings make a real difference.

### Change the default admin password
If you've never changed it, do it now. Default credentials are publicly listed for every router model. Anyone on your network (or even sometimes from outside) can log in with them.

### Use WPA3 or WPA2 encryption
Log into your router admin page and check the WiFi security settings. You want:
- **WPA3** (best, if your router supports it)
- **WPA2 (AES)** (good, widely supported)

Avoid **WEP** and **WPA (TKIP)**, these are outdated and broken.

### Disable WPS
WiFi Protected Setup sounds convenient but has known security vulnerabilities. Turn it off in your router settings if you see it.

### Keep router firmware updated
Manufacturers release firmware updates that patch security vulnerabilities. Log into your router admin page and check for updates. Some routers do this automatically, most don't.

### Consider a guest network
If you have visitors or IoT devices (smart bulbs, cameras, etc.), put them on a separate guest network. That way, even if one device is compromised, it can't reach your main devices.

### Disable remote management
Most home routers don't need to be managed from outside your home network. Turn off remote management in your router settings unless you specifically need it.

---

## 10. Additional macOS Security Tips

Beyond the network scan, here are things worth checking on your Mac itself.

### Enable the Firewall

macOS has a built-in firewall that's off by default on some versions.

Go to: **System Settings > Network > Firewall** and turn it on.

You can also enable "Stealth Mode" which makes your Mac not respond to ping requests from the network, useful if you're on public WiFi.

### Check Sharing Services

Open **System Settings > General > Sharing** and turn off anything you don't actively use. Services like Screen Sharing, File Sharing, and Remote Login create open ports on your machine.

### Use a VPN on Public WiFi

On public networks (cafes, airports, hotels), your traffic can be intercepted. A VPN encrypts it. There are many options, paid ones are generally more trustworthy than free ones.

### Keep macOS Updated

Security patches come with system updates. Go to **System Settings > General > Software Update** and enable automatic updates, or at least check regularly.

### Check for Suspicious Login Items

Open **System Settings > General > Login Items & Extensions**. Review what's set to launch at startup. Remove anything you don't recognize.

### Monitor Network Activity (Advanced)

If you want to see what your Mac is actually sending out in real time, you can use the built-in Activity Monitor:

Open **Activity Monitor > Network tab**. It shows which processes are sending and receiving data.

For deeper inspection, `Little Snitch` (paid) or `Lulu` (free) are firewall apps that notify you when any app tries to connect to the internet.

---

## 11. Quick Reference Cheat Sheet

```bash
# Find your IP and network range
ifconfig en0 | grep inet

# Ping scan (find all devices on network)
sudo nmap -sn 192.168.x.0/24

# Service scan on a specific device
sudo nmap -sV 192.168.x.101

# Force scan (skip ping check)
sudo nmap -Pn -sV 192.168.x.101

# Ping a device manually
ping -c 3 192.168.x.101

# Check ARP table (recently seen devices)
arp -a
```

**MAC address lookup:** [maclookup.app](https://maclookup.app)

**Common router admin addresses:**
- `192.168.1.1`
- `192.168.0.1`
- `192.168.254.254`
- `10.0.0.1`

---

## Contributing

Found an error or want to add something? Pull requests are welcome. Keep it practical and jargon-light where possible.

## License

MIT License. Use freely, credit appreciated.
