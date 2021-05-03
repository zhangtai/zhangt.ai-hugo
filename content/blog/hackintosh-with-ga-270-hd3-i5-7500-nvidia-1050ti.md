---
title: "Install High Sierra on GA-Z270 HD3 with Intel i5-7500 CPU and NVIADIA 1050 TI Graphic Card"
date: 2018-02-18T10:20:48+08:00
---

> update 2018-02-21

## Preinstallation

Follow the [Guide](https://www.tonymacx86.com/threads/unibeast-install-macos-high-sierra-on-any-supported-intel-based-pc.235474/)

I had a problem when writing image into USB disk, it was saying that writing EFI file failed, format the usb disk with below command resolved the issue:

``` sh
sudo diskutil partitionDisk /dev/disk2 GPT JHFS+ hackingtosh-disk R
```

When using UniBeast, don't inject graphic card options.

## Connections

I was connecting from DP port of Graphic Card to Monitor/TV

## BIOS setting for Gigabyte 200

Change the following settings:

### Save & Exit :

- Load Optimized Defaults

### M.I.T.

- Advanced Memory Settings → Extreme Memory Profile(X.M.P.) : Profile1

### BIOS

- Fast Boot : Disabled
- Windows 8/10 Features : Other OS
- LAN PXE Boot Option ROM : Disabled
- Storage Boot Option Control : UEFI

### Peripherals

- Initial Display Output: IGFX | PCIe 1 Slot
- Super IO Configuration → Serial Port : Disabled
- Network Stack Configuration → Network Stack : Disabled
- USB Configuration → XHCI Hand-off : Enabled

### Chipset

- Vt-d : Disabled
- Wake on LAN Enable : Disabled

Press F10 to Save and Exit the BIOS

Press F12 load into boot menu and select USB disk: UEFI OS

## Installing

1. Plug the usb disk into one of **USB 2.0** interface of your board
2. Select USB boot option
3. open Disk Utility and click View to show all disk
4. select the disk and erase, instead of select the partition within the disk
5. erase disk with format **APFS**
6. Installing with some reboots, first one takes no more than 5 mins, since Clover is not installed yet, press F12 after each reboot and select "Boot macOS Install from HS(my disk name)" to continue the installation
7. Continue the installation, takes about 15 mins. You will have some reboots, even hard reboot manually...

## Post installation

- Install Multibeast. Here is my options:

    - Quick Start: UEFI
    - Drivers: Audio(Universal - VoodooHDA v2.9.0d10), Network(Intel - IntelMausiEthernet v2.3.0), USB(Increase Max Port Limit 200 Series)
    - Bootloaders: Clover UEFI Boot Mode + Emulated NVRAM
    - Customize: no change

- Install webdriver of NVIADIA but not the latest version, use [nvidia-update](https://github.com/Benjamin-Dobell/nvidia-update) instead
- Enable 4k@60ghz with [SwitchResX](http://www.madrau.com/)
