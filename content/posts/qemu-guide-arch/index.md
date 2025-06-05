---
title: "QEMU Setup Guide for (Arch) Linux"
date: 2025-06-05T09:25:00+07:00
draft: true
tags: ["Guide", "Linux", "Virtualization", "QEMU"]
categories: ["Guide", "Linux"]
summary: "My clumsy QEMU setup guide based on how I personally tackled it."
---
I have been using VirtualBox for the longest time when it comes to virtual machines, and never really felt like I needed anything more functionally. The only minor "issue" that I really noticed was the performance (boot time, latency, etc.) that I quickly dismissed once I got used to it. This is where QEMU comes into the picture, as it has existed as an alternative that offers superior performances, but at the cost of a more complicated setup process. For that reason, I had regrettably put it on the back burner for a while, until I finally gave it a try and was very surprised at how pleasant the installation process was. This guide was made mostly as a self-reminder, but also for anyone who are interested in trying QEMU and are looking for a reasonably simple guide to get started.

**Disclaimer**: I'm still a Linux beginner as of the time of writing, so constructive criticisms are greatly appreciated. If anything, this should serves as a supplement to better, official documentations such as Arch Linux's wiki for [QEMU](https://wiki.archlinux.org/title/QEMU) & [Virt-manager](https://wiki.archlinux.org/title/Virt-manager), not to replace them.

## Installation
Install the required packages: `qemu-desktop` has only x86_64 emulation by default which is what I used, but can be replaced with `qemu-full` for the entire suite. For a minimal install, replace with `qemu-base`, but will require extra manual configurations for QXL/SPICE which will not be covered in this guide.
```sh
sudo pacman -S qemu-desktop virt-manager virt-viewer dmidecode dnsmasq bridge-utils iptables vde2
```

```sh
sudo systemctl enable libvirtd.service
```
## Setting 
