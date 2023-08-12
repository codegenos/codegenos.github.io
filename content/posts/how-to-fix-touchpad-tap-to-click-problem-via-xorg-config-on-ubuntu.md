---
author: "CodeGenos"
title: "How to Fix Touchpad Tap to Click Problem via Xorg Configuration File on Ubuntu"
date: 2023-08-11T07:32:05+03:00
description: "Touchpad tap-to-click sometimes is not working when you first installed Ubuntu linux. You can fix this problem via adding Xorg configuration file for touchpad."
tags: ["Linux", "Ubuntu", "Touchpad", "Xorg", "Libinput"]
categories: ["Linux"]
ShowToc: true
ShowBreadCrumbs: true
draft: false
---

## Problem
Touchpad tap-to-click was not working when i first installed Ubuntu linux. I will show you how to fix this problem.

Xorg is responsible for managing input devices like touchpads. It uses libinput software library for input handling. So let's learn some information about Xorg and libinput:

### Xorg
>Xorg, also known as X.Org Server, is an open-source implementation of the X Window System, commonly referred to as "X11" or simply "X." Xorg serves as the display server for graphical environments on Linux and other Unix-like systems. It manages input and output devices such as keyboards, mice, touchpads, monitors, and graphics hardware. You can read more at <a href="https://www.x.org/wiki/" target="_blank">Xorg</a> wiki page.

### Libinput
>Libinput is an open-source software library that provides a unified input handling interface for Linux-based systems, particularly those using the X Window System (Xorg) or Wayland display server protocols. It is designed to manage input devices such as keyboards, mice, touchpads, and other input peripherals in a consistent and efficient manner.

>The primary purpose of libinput is to abstract the complexities of dealing with various input devices and to provide a common API that applications and desktop environments can use to access input data. This abstraction allows developers to create user-friendly interfaces and interactions without needing to worry about the specific details of different hardware devices.

### Check input devices
One way to check which devices are managed by libinput is the xorg logfile. To view the devices, run the follwing command:

```bash
grep -e "Using input driver 'libinput'" /var/log/xorg.conf.d/Xorg.0.log
```

>```[   181.467] (II) Using input driver 'libinput' for 'Power Button'
>[   181.523] (II) Using input driver 'libinput' for 'Video Bus'
>[   181.604] (II) Using input driver 'libinput' for 'Integrated Camera: Integrated C'
>[   181.651] (II) Using input driver 'libinput' for 'Logitech USB Optical Mouse'
>[   181.700] (II) Using input driver 'libinput' for 'Ideapad extra buttons'
>[   181.734] (II) Using input driver 'libinput' for 'MSFT0001:00 2808:0101 Touchpad'
>[   181.841] (II) Using input driver 'libinput' for 'MSFT0001:00 2808:0101 Mouse'
>[   181.896] (II) Using input driver 'libinput' for 'AT Translated Set 2 keyboard'
>```

You can see the touchpad device log:

`[ 181.734] (II) Using input driver 'libinput' for 'MSFT0001:00 2808:0101 Touchpad'`

## Fix
### Tapping button re-mapping

Create `30-touchpad.conf` file under `etc/X11/xorg.conf.d/`

```bash
sudo nano /etc/X11/xorg.conf.d/30-touchpad.conf
```

Paste the following text and save:

```
Section "InputClass"
    Identifier "touchpad"
    Driver "libinput"
    MatchIsTouchpad "on"
    Option "Tapping" "on"
    Option "TappingButtonMap" "lmr"
EndSection
```

>Option "Tapping" "on": tapping a.k.a. tap-to-click
>Option "TappingButtonMap" "lmr": left/middle/right buttons

After that the problem will be fixed. Now you will be able to click when you tap touchpad.