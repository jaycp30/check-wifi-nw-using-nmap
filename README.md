# macOS WiFi Security Guide

A practical, hands-on guide for auditing your home or office WiFi network using your Mac. No prior security experience needed, but it doesn't talk down to you either.

## What this covers

- Installing and using `nmap` to scan your local network
- Identifying every device connected to your WiFi
- Spotting devices that don't belong
- Understanding MAC addresses and what they tell you
- Hardening your router and WiFi settings
- Additional macOS-specific security tips

## Who this is for

Anyone on a Mac who wants to actually understand what's on their network, not just trust that everything is fine. Whether you're a complete beginner or someone who's comfortable in Terminal, this guide has something useful for you.

## Files

| File | Description |
|------|-------------|
| `README.md` | This file, repo overview |
| `GUIDE.md` | Full step-by-step security guide |

## Quick Start

If you just want to run a scan right now:

```bash
# Install nmap via Homebrew
brew install nmap

# Find your network range
ifconfig en0 | grep "inet "

# Run a ping scan (replace with your actual range)
sudo nmap -sn 192.168.1.0/24
```

See [GUIDE.md](./GUIDE.md) for the full walkthrough and how to interpret results.

## Prerequisites

- A Mac (macOS 12 Monterey or later recommended, most steps work on older versions too)
- [Homebrew](https://brew.sh) installed, or willing to install it
- Access to your router's admin page (helpful but not required)
- Terminal, which is already on your Mac

## Disclaimer

This guide is for auditing networks you own or have explicit permission to scan. Scanning networks you don't own is illegal in most jurisdictions. Use responsibly.

---

Contributions and corrections welcome via pull request.
