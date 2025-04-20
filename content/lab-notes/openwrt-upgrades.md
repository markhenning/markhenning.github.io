---
title: OpenWRT Upgrades
draft: false
---
# Upgrade Method

OpenWRT use of an overlay filesystem leads to some interesting options for it's upgrade path. Whilst it's possible to upgrade individual packages on a device in a pinch to fix a CVE, OpenWRT generally prefers a different method - rolling a complete firmware bundle.

## Making a Package List

This actually turns out to be a lot easier than it sounds, in fact it's just a case of knowing what you need in there. On a new device, this is easy, it's just the default list, no changes. If it's an older device, you can get a list of packages from:

```
/usr/lib/opkg/status
```

This contains everything installed, you'll need a list of the packages, but not any that are "Auto Installed" (these get handled as part of the base image or are pre-reqs for other packages that aren't auto-installed)

![[Pasted image 20250409151521.png]]

## Building the Image

The package can be built and downloaded from a tool on the OpenWRT site:

[OpenWRT Firmware Selector](https://firmware-selector.openwrt.org/)

Before you start, you'll need the model of your current device, I've got a pair of WSM20s and a Cudy v3000. I'll work with the WSM20:


| Step | Task                                                                                                                                                                                                               | Screenshot                           |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------ |
| 1    | Select your hardware from the list                                                                                                                                                                                 | ![[Pasted image 20250409152720.png]] |
| 2    | Select the firmware you want to run.<br><br>(Top of the list is the latest - however "SNAPSHOT" is the current daily build - don't user this unless told to!)                                                      | ![[Pasted image 20250409152754.png]] |
| 3    | Expand "Customize installed packages....." to show the list of packages to be installed in your firmware                                                                                                           | ![[Pasted image 20250409152924.png]] |
| 4    | Customise the list to add in the packages you need <br><br>(If you removed default packages from the running firmware, do so again - e.g. I remove wpad-basic-mbedtls because I replace it with wpad-mesh-mbedtls) | ![[Pasted image 20250409153350.png]] |
| 5    | Click Request Build and wait                                                                                                                                                                                       | ![[Pasted image 20250409153556.png]] |
| 6    | Wait for progress as it builds. <br>The package list will be validated for conflicts before building                                                                                                               | ![[Pasted image 20250409153617.png]] |
| 7    | When complete, the "SYSUPGRADE" download link will appear, click and download the file                                                                                                                             | ![[Pasted image 20250409153718.png]] |
That's it, just need to install it now.

The most likely place you'll have problems is two places:
1. If the firmware is brand new (less than 24 hours) - it may still be seeding to various build nodes and you might get spurious errors)
2. If you can't solve a package conflict, remove the ones you added that are preventing the build. The image you get will work fine, but will not support the features you need until you install that feature later - this is very easy to do

## Installing the Firmware

Easiest way is through the web GUI:

|     | Task                                                           | Screenshot                           |
| --- | -------------------------------------------------------------- | ------------------------------------ |
| 1   | Log into the management interface                              |                                      |
| 2   | Navigate "System -> Backup/Flash Firmware"                     | ![[Pasted image 20250409155757.png]] |
| 3   | Make a backup with "Generate archive"                          | ![[Pasted image 20250409155919.png]] |
| 4   | Click "Flash Image"                                            | ![[Pasted image 20250409155950.png]] |
| 5   | "Browse..." to select the image<br><br>Click "Upload"          | ![[Pasted image 20250409160053.png]] |
| 6   | Wait for the upload to complete                                |                                      |
| 7   | Tick the box to "Include a list of packages"                   |                                      |
| 8   | Click "Continue" to start the upgrade                          | ![[Pasted image 20250409160233.png]] |
| 9   | Wait for the upgrade to complete and the router to come back up |                                      |

## My Personal Package List

For my own notes, this is the list I use to compile the firmware for my network:

Packages to Remove

```
wpad-basic-mbedtls
```

Packages to Add

```
wpad-mesh-openssl wpad-mesh-openssl acme acme-acmesh luci-app-acme acme-acmesh-dnsapi kmod-batman-adv luci-proto-batman-adv
```




