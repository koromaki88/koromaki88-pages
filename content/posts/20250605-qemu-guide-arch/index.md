---
title: "QEMU Setup Guide for (Arch) Linux"
date: 2025-06-05T09:25:00+07:00
draft: false
tags: ["Arch", "Linux", "Virtualization", "QEMU"]
categories: ["Guides"]
summary: "My clumsy QEMU setup guide based on how I personally tackled it."
---
I have been using VirtualBox for the longest time when it comes to virtual machines, and never really felt like I needed anything more functionally. The only minor "issue" that I really noticed was the performance (boot time, latency, etc.) that I quickly dismissed once I got used to it. This is where QEMU comes into the picture, as it has existed as an alternative that offers superior performances, but at the cost of a more complicated setup process. For that reason, I had regrettably put it on the back burner for a while, until I finally gave it a try and was very surprised at how pleasant the installation process was. This guide was made mostly as a self-reminder, but also for anyone who are interested in trying QEMU & Virt-manager and are looking for a reasonably simple guide to get started.

**Disclaimer**: I'm still a Linux beginner as of the time of writing, so constructive criticisms are greatly appreciated. If anything, this should serves as a supplement to better, official documentations such as Arch Linux's wiki for [QEMU](https://wiki.archlinux.org/title/QEMU) & [Virt-manager](https://wiki.archlinux.org/title/Virt-manager), not to replace them.

## Installation
Install the required packages. On Arch, [qemu-desktop](https://archlinux.org/packages/extra/x86_64/qemu-desktop/) has only x86_64 emulation by default which is what I used, but can be replaced with [qemu-full](https://archlinux.org/packages/extra/x86_64/qemu-full/) for the entire suite. For a minimal install, replace with [qemu-base](https://archlinux.org/packages/extra/x86_64/qemu-base/), but will require extra manual configurations for QXL/SPICE which will not be covered in this guide.
```sh
sudo pacman -S qemu-desktop virt-manager virt-viewer dmidecode dnsmasq bridge-utils iptables vde2 openbsd-netcat
```

## Setup
[Libvirt](https://wiki.archlinux.org/title/Libvirt) is the toolkit for managing virtualization platforms and is required for QEMU. Enable & start `libvirtd.service`, which will consequently do the same for `libvirtd.socket`.
```sh
sudo systemctl enable --now libvirtd.service
```

Add the current user to the `libvirt` group, otherwise Virt-manager will throw an error when trying to make a connection.
```sh
sudo usermod -a -G libvirt $(whoami)
newgrp libvirt
```

With this, you are ready to use QEMU! There are a few things you want to ensure are properly configured to everything to work:
- QEMU/KVM > Details > Virtual Networks - Ensure `virbr0` is Active (enable it if it isn't).jj
- If you have separate root & home partitions, it is recommended to add a new pool and set any directory in home for your VM disks to avoid using up root disk space. Go to QEMU/KVM > Details > Storage, click 'Add Pool' at the bottom left corner and change 'Target path' as desired.
- As best practice suggested by the Arch wiki, ensure external directories outside the default pool are owned by `libvirt-qemu` group:
```
chown "$USER":libvirt-qemu /path/to/vm/folder
```

To create a new VM, just click the first icon in the toolbar and configure the settings as you would on other virtualization platforms.

## Troubleshoot
- **VPN Blocking Connection**: One problem I had was `qemu:///system` not being able to establish a connection since I was using a VPN. Libvirt's `virbr0` expects to route traffic from a specific subnet (which was `192.168.122.0/24` by default for me) through my actual internet connection, not my VPN tunnel. On Mullvad VPN, this can be fixed by enabling **local network sharing** (allowing network communication outside the tunnel to local ranges) or **split tunneling** (bypass the VPN tunnel entirely). These options may be different or unavailable on other VPN providers.
- **Firewall (UFW) Blocking Connection**: Same problem as above, but with UFW (Uncomplicated Firewall) this time. A fresh Arch Linux setup should not have UFW, and upon install the firewall will block all **incoming** traffic by default. For the VM to communicate with our host, we need to allow traffic between `virbr0` and the host machine by adding the following rules & :
```sh
sudo ufw allow in on virbr0
sudo ufw route allow in on virbr0 out on wlan0  
# change wlan0 to your network interface (check with `ip addr`)
# or wg0-mullvad for Mullvad VPN
```
An alternative to `ufw route` would be to just enable forwarding policy by default (from `DROP` to `ACCEPT`), though I prefer to keep default configurations and add individual rules instead.
```sh
/etc/default/ufw
DEFAULT_FORWARD_POLICY="ACCEPT"
```
