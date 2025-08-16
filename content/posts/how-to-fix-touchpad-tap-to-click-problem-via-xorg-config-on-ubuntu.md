---
author: "CodeGenos"
title: "Ubuntu: Fix Touchpad Tap-to-Click via Xorg (libinput)"
date: 2023-08-11T07:32:05+03:00
description: "Tap-to-click not working on Ubuntu? Enable it in GNOME or with an Xorg libinput config (30-touchpad.conf). Steps, commands, and verification."
tags: ["Linux", "Ubuntu", "Touchpad", "Xorg", "Libinput"]
categories: ["Linux"]
ShowToc: true
ShowBreadCrumbs: true
draft: false
---

## Problem
Tap-to-click on the touchpad was not working after I installed Ubuntu. Here are quick and advanced fixes.

Note: This guide’s fix targets Xorg. Under Xorg, the libinput driver reads options from xorg.conf.d, so we can enforce Tapping system-wide. On Wayland, these options are managed by GNOME; use the GUI toggle instead—Xorg configs do not apply to Wayland sessions.

## TL;DR / Quick start
- Wayland/GUI: Settings → Mouse & Touchpad → enable "Tap to Click".
- Xorg (terminal): create config and enable tapping, then re-login.

```bash
# Create config dir and write touchpad config
sudo install -d /etc/X11/xorg.conf.d/
sudo tee /etc/X11/xorg.conf.d/30-touchpad.conf >/dev/null <<'EOF'
Section "InputClass"
    Identifier "libinput touchpad"
    MatchIsTouchpad "on"
    MatchDriver "libinput"
    Driver "libinput"
    Option "Tapping" "on"
    Option "TappingButtonMap" "lmr"
EndSection
EOF

# Reboot or log out/in, then verify
grep -i "Using input driver 'libinput'" /var/log/Xorg.0.log || \
  grep -i "Using input driver 'libinput'" ~/.local/share/xorg/Xorg.0.log || \
  sudo journalctl -b _COMM=Xorg | grep -i "Using input driver 'libinput'"
```

## Quick fix (GUI/Wayland)
If you use Wayland or the default GNOME session:
- Open Settings → Mouse & Touchpad
- Enable "Tap to Click"

Note: Xorg-specific configs below do not affect Wayland sessions. If you want to use the Xorg method, log out and choose "Ubuntu on Xorg" on the login screen.

## Check input devices (Xorg)
One way to see which devices are managed by libinput is the Xorg log file. Run:

```bash
grep -i "Using input driver 'libinput'" /var/log/Xorg.0.log || \
  grep -i "Using input driver 'libinput'" ~/.local/share/xorg/Xorg.0.log || \
  sudo journalctl -b _COMM=Xorg | grep -i "Using input driver 'libinput'"
```

Example output:

```
[   181.467] (II) Using input driver 'libinput' for 'Power Button'
[   181.523] (II) Using input driver 'libinput' for 'Video Bus'
[   181.604] (II) Using input driver 'libinput' for 'Integrated Camera: Integrated C'
[   181.651] (II) Using input driver 'libinput' for 'Logitech USB Optical Mouse'
[   181.700] (II) Using input driver 'libinput' for 'Ideapad extra buttons'
[   181.734] (II) Using input driver 'libinput' for 'MSFT0001:00 2808:0101 Touchpad'
[   181.841] (II) Using input driver 'libinput' for 'MSFT0001:00 2808:0101 Mouse'
[   181.896] (II) Using input driver 'libinput' for 'AT Translated Set 2 keyboard'
```

You can see the touchpad device line, for example:
`[ 181.734] (II) Using input driver 'libinput' for 'MSFT0001:00 2808:0101 Touchpad'`

## Fix via Xorg config (advanced)
### Enable tapping and button mapping
Create `30-touchpad.conf` under `/etc/X11/xorg.conf.d/` (create the folder if needed):

```bash
sudo install -d /etc/X11/xorg.conf.d/
sudo nano /etc/X11/xorg.conf.d/30-touchpad.conf
```

Paste and save:

```
Section "InputClass"
    Identifier "libinput touchpad"
    MatchIsTouchpad "on"
    MatchDriver "libinput"
    Driver "libinput"

    # Enable tap-to-click
    Option "Tapping" "on"

    # Left/Middle/Right mapping for 1/2/3-finger taps
    Option "TappingButtonMap" "lmr"
EndSection
```

Apply the change by logging out/in or rebooting so Xorg reloads the config.

## Verify the change
- In Xorg, re-check the Xorg log for your touchpad line and confirm libinput is used (see command above).
- Or check device capabilities:

```bash
libinput list-devices | sed -n '/touchpad/,+20p' | sed -n '1,20p'
```

Look for `Tapping: enabled`.

## More information
- Xorg manages input devices and can be configured via `xorg.conf.d` snippets. See the man page: https://www.x.org/releases/current/doc/man/man5/xorg.conf.5.xhtml
- libinput provides unified input handling across Xorg and Wayland. Tapping docs: https://wayland.freedesktop.org/libinput/doc/latest/tapping.html

## FAQs
- How do I enable tap-to-click on Wayland?
  - Use GNOME Settings → Mouse & Touchpad → enable Tap to Click. Xorg config files do not apply to Wayland.
- Where is the Xorg log file?
  - Typically `/var/log/Xorg.0.log`. On newer systems it may be `~/.local/share/xorg/Xorg.0.log` or available via `journalctl -b _COMM=Xorg`.
- What does `TappingButtonMap "lmr"` do?
  - Maps 1/2/3-finger taps to left/middle/right clicks respectively.