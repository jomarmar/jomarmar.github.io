---
layout: post
title: "Archlinux: install Gnome (minimal)"
date: "2015-12-16 13:56"
categories: "os"
tags: "archlinux linux"
about: "How to perform a minimal Gnome installation in Archlinux."
---

## Install Gnome in Archlinux

To install an empty Gnome desktop install the following packages

```
# pacman -S xorg-xinit xorg-server gnome-session gdm
# cp /etc/X11/xinit/xinitrc $HOME/.xinitrc
```

Edit `$HOME/.xinitrc` and replace the following lines:

```
twm &
xclock -geometry 50x50-1+1 &
xterm -geometry 80x50+494+51 &
xterm -geometry 80x20+494-0 &
exec xterm -geometry 80x66+0+0 -name login
```

with

```
exec gnome-session
```

## Install Intel Graphics HD 5000

Configure Graphics Card: Intel Graphics HD 5000

```
# pacman -S xf86-video-intel xf86-video-fbdev xf86-video-vesa libva-intel-driver libva libvdpau-va-gl
```
Create a file: `/etc/modprobe.d/i915.conf` with the following line:

```
options i915 modeset=1
```
Create a file: `/etc/X11/xorg.conf.d/20-intel.conf` with the following content:

```
Section "Device"
  Identifier "Card0"
  Driver "intel"
  VendorName "Intel Corporation"
  BusID "PCI:0:2:0"
EndSection
```

Initramfs:

Edit the following lines in `/etc/mkinitcpio.conf`

```
MODULES="intel_agp i915"
FILES="/etc/modprobe.d/i915.conf"
HOOKS="base udev autodetect modconf block lvm2 filesystems keyboard fsck"
```
Then regenerate the Initramfs image:

```
# mkinitcpio -p linux
```

## Other useful packages for Gnome

* gnome-contro-panel
* gnome-terminal
* nautilus
* gedit
* gvfs-smb
* gvfs-nfs
* gvfs-afc
* gvfs-afp
* gvfs-gphoto2
