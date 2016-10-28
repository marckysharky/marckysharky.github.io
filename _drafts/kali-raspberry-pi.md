---
layout: post
title: Kali Linux on Raspberry Pi 3
---

This setup will take you through how to install Kali Linux to an SD Card, using a Mac, for use with a Raspberry Pi.

### Requirements
- SD Micro card, at least `8GB` capacity

### Instructions
Download the [latest Kali image](https://www.offensive-security.com/kali-linux-arm-images/), from the RaspberryPi Foundation section:

```
wget https://images.offensive-security.com/arm-images/kali-2.1.2-rpi2.img.xz
```
Insert the SD card into the machine, and open a terminal window.

Find the corresponding device identifier, e.g. `disk2`:

```
diskutil list
```

Extract the `img` file from the downloaded `xz`:

```
brew install xz
xz -d kali-2.1.2-rpi2.img.xz
```

Unmount and write the image to the SD card (`sudo` maybe required):

```
diskutil unmountDisk /dev/disk2s1
dd bs=1m if=kali-2.1.2-rpi2.img of=/dev/rdisk2
```
This will take a few minutes.

Eject the SD card (`sudo` maybe required)

```
sudo diskutil eject /dev/rdisk2
```

Your SD card should now be ready to use with your Raspberry Pi. The default configuration for login is `root:toor`.

### Script
**Use this script with extreme caution.**

<script src="https://gist.github.com/marckysharky/34d131075448e559ab15f81d840871fe.js"></script>

### References
- [Kali Latest Images](https://www.offensive-security.com/kali-linux-arm-images/)
- [Kali Linux - Raspberry Pi2](http://docs.kali.org/kali-on-arm/kali-linux-raspberry-pi2)
- [Installing Operating System Images on Mac OS - Raspberry Pi](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md)
- [Gist](https://gist.github.com/marckysharky/34d131075448e559ab15f81d840871fe)
