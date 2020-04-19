---
layout: post
title:  "Linux on Chromebox"
excerpt: "Installing GalliumOS on a HP Chromebox"
date:   2020-04-19 09:16:00
---

I wanted a small x86 machine to have running 24/7 with some services on it as my Raspberry Pi 3 was a little underpowered. I looked around on ebay and while NUCs still command higher prices I saw some Chromeboxes available. I ended up with a HP Chromebox G1 (HP Chromebox CB1-020NA) for £54. I'm happy with this, the exact same price as a 4GB Raspberry Pi 4 but more flexible and powerful, also includes it's own power adapter and storage.

| | |
|--|--|
| CPU | [Intel Celeron 2955U 2C @1.4GHz](https://ark.intel.com/content/www/us/en/ark/products/75608/intel-celeron-processor-2955u-2m-cache-1-40-ghz.html) |
| RAM | 4GB DDR3L |
| GPU | Intel HD graphics 4400 |
| Disk | 16GB SSD |
| Price | £54 |

My first searches for Linux on a Chromebook pointed to [Crouton](https://github.com/dnschneid/crouton), but this is running ChromeOS alongside Linux which was not what I wanted. Further searched brought up [GalliumOS](https://galliumos.org/) which is full Linux based on Ubuntu 18.04 but customised specifically for ChromeOS hardware. There is another dual booting option called [chrx](https://chrx.org/) but as that's not what I wanted I also ignored it.

Importantly the HP machine is listed on the [Hardware Compatibility](https://wiki.galliumos.org/Hardware_Compatibility) for GalliumOS. Knowing the hardware codename `ZAKO` helped when looking through the documentation as this is how the site refers to all machines.

Alongside GalliumOS I would be using [MrChromebox.tech](https://mrchromebox.tech) for custom firmware and utilities.
Strictly speaking custom firmware may not have been needed but was listed as recommended. It corrects a listed [factory bug](https://wiki.galliumos.org/Firmware#Factory_Bugs) in the original firmware with USB ports in the alternative legacy boot mode way to install Linux.

## Installing Linux

### Disable firmware write protect

The machine needed to be put in to developer mode to enable the writing of custom firmware and installation of the new OS. For this machine this required removing an internal screw (easy). I found a guide for this and a picture on [kodi.wiki](https://kodi.wiki/view/Chromebox#Device_Preparation).

### Enable developer mode

A guide for this is provided by GalliumOS wiki. `ZAKO` shares it's hardware with an Asus machine `PANTHER` so you can follow the [instructions](https://wiki.galliumos.org/Installing/Panther#Enable_Developer_Mode_and_Boot_Flags) for it.

### Write custom firmware

MrChromebox provides a [script](https://mrchromebox.tech/#fwscript) which installs custom firmware.
In ChromeOS guest mode I opened a terminal with `CTRL+ALT+T` and entered `shell`. Then downloaded and ran the script:

```bash
cd; curl -LO https://mrchromebox.tech/firmware-util.sh && sudo bash firmware-util.sh
```

I chose option `3`
```bash
3) Install/Update Full ROM Firmware
```

After that it was straight forward to go through the few menu screens. I took the advice from the utility to backup my old firmware. This only required popping in a USB drive for it to write to, this failed at first but after a reformat of the drive to FAT32 it succeeded.

### Source iso and create bootable USB

Downloaded the correct `.iso` file from the [downloads](https://galliumos.org/download) page of GalliumOS. `Haswell` for this machine.

I tried [rufus](https://rufus.ie/) at first but event with the correct settings (`GPT`, `UEFI`, `DD`) this was failing to boot for me. Instead I used [Etcher](https://www.balena.io/etcher/) which worked first time without any setting changes required. The wiki has a [guide](https://wiki.galliumos.org/Installing/Creating_Bootable_USB#On_Windows.2C_macOS_.28OS_X.29.2C_and_Linux_.28using_Etcher.29) if needed.

### Install GalliumOS

Getting in to the [UEFI](https://mrchromebox.tech/#bootmodes) of the custom firmware requires the pressing of any key within ~1 second of powering the machine on. Then the boot order can be changed to use your USB first. After that the install process is as easy as any Ubuntu type system.

## Summary

I did find some of the back and forth between GalliumOS and Mr.Chromebox documentation a little jarring, leaving me somewhat concerned I might to brick the device. Ultimately after double checking my notes running through all the steps went smoothly.

GalliumOS itself has been great. It boots up fast and uses small amounts of RAM and the limited 16GB SSD. WIFI and Bluetooth both worked out of the box.
