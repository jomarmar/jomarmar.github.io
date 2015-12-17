---
layout: post
title: "Archlinux: initial setup and installation"
date: "2015-12-16 13:07"
categories:  os
tags: linux archlinux
about: "Installation of Archlinux on an Intel i7 laptop with 8 GB RAM, 120 GB SSD and
Intel Graphics HD 5000."
---

## Hardware Description

**Model:** [Slimbook](https://slimbook.es) 715-Intel i7

**CPU:** Intel i7 - 4550U, 1.5GHz, Turbo Boost 3.0, 2 Core 4 Threads, 4M Cache [+info](https://slimbook.es/tutoriales/slimbook/41-comparativa-procesador-i7)

**Graphics Card:** Intel Graphics HD 5000

**RAM:** 8GB

**SDD:** 120GB

**Display:** 13.3" FullHD 1920x1080px LED

**Wireless LAN:** Intel A/C Dual Band 802.11 b/g/n

**Bluetooth:** 4.0

**Camera:** 2mp

**Others:**

-  2 USB 3.0 ports
-  Mini-HDMI
-  SD/MMC
-  External RJ45 (LAN) Adapter

## Archlinux OS

### Preparation

Follow [this](https://wiki.archlinux.org/index.php/Category:Getting_and_installing_Arch) for instructions on downloading the installation medium and methods for booting it to the target machine.

### Network

Usually network works out-of-the-box when booting from the installation medium. Nevertheless, if problems are found:

#### Wireless

Retrieve available network interfaces with:

```
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp2s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether 00:16:96:e3:13:f2 brd ff:ff:ff:ff:ff:ff
3: wlp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether b4:6d:83:9a:9e:53 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.132/24 brd 192.168.0.255 scope global dynamic wlp3s0
       valid_lft 84162sec preferred_lft 84162sec
    inet6 fe80::b66d:83ff:fe9a:9e53/64 scope link
       valid_lft forever preferred_lft forever
```
In this case the wireless interface is **wlp3s0**.

Run the following command to setup configure the wireless connection:

```
# wifi-menu <interface>
```

#### Wired
It should be enough to start the DHCP client as:

```
# systemctl restart dhcpd.service
```

### Storage devices (Partitions & LVM)

List all devices (hdd) available and make sure you choose the right one:

```
# lsblk
NAME                 MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                    8:0    0 119.2G  0
```

Create one partition for **boot** systems and another for data/swap/root system.

```
# parted /dev/sdx
(parted) mklabel gpt
```
Set up **boot** partition
```
(parted) mkpart ESP fat32 1MiB 513MiB
(parted) set 1 boot on
```
Set up data partition
```
(parted) mkpart primary ext4 513MiB 100%
```

The next step is to format the partitions:

**boot** partition:
```
# mkfs.fat -F32 /dev/sdxY
```
**data** partition:
```
# mkfs.ext4 /dev/sdxY
```

Now it is turn to setup the LVM.

#### LVM Set up

Three logical volumes will be created for:
* root
* user's data
* swap [optional]

```
# pvcreate /dev/sdax
# vgcreate VolGroup00 /dev/sdax
# lvcreate -L 25G VolGroup00 -n lroot
# lvcreate -L 8G VolGroup00 -n lswap
# lvcreate -l +100%FREE VolGroup00 -n lhome
#
# modprobe dm_mod
# vgscan
# vgchange -ay
```

Format the logical volumes

```
# mkfs.ext4 /dev/mapper/VolGroup00-lroot
# mkfs.ext4 /dev/mapper/VolGroup00-lhome
# mkswap /dev/mapper/VolGroup00-lswap
# swapon /dev/mapper/VolGroup00-lswap
```


### Mount and create basic installation

```
# mount /dev/mapper/VolGroup00-lroot /mnt
# mkdir /mnt/boot
# mkdir /mnt/home
# mount /dev/sdax /mnt/boot
# mount /dev/mapper/VolGroup00-lhome /mnt/home
```

1.  Install base system:

    ```
    # pacstrap /mnt base base-devel
    ```

2.  Generate fstab:

    ```
    # genfstab -U /mnt > /mnt/etc/fstab
    ```

3.  Configure the base system:

    ```
    # arch-chroot /mnt /bin/bash
    ```

4.  Set Locale: Uncomment `en_US.UTF-8` and `es_ES.UTF-8` from `/etc/locale.gen` and run:

    ```
    # locale-gen
    # echo LANG=en_US.UTF-8 > /etc/locale.conf
    # echo KEYMAP=es > /etc/vconsole.conf
    ```

5.  Set Time:

    ```
    # ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
    # hwclock --systohc --utc
    ```

6.  Configure Graphics Card: Intel Graphics HD 5000

    ```
    # pacman -S xorg-xinit xorg-server xf86-video-intel xf86-video-fbdev xf86-video-vesa libva-intel-driver libva libvdpau-va-gl [gnome-session gdm gnome-control-panel gnome-terminal nautilus]
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

7.  Initramfs:

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

8.  Finally create a boot loader (EFI/GPT):

    ```
    # bootctl install
    ```

9.  Create a boot entry in `/boot/loader/entries/arch.conf`:

    ```
    title 	Arch Linux
    linux 	/vmlinuz-linux
    initrd	/initramfs-linux.img
    options 	root=/dev/mapper/VolGroup00-lroot rw
    ```

10. Choose the default entry on loader

    ```
    # vi /boot/loader/loader.conf
    ```
Append these lines:

    ```
    timeout 3
    default arch
    ```

11. Set the hostname:

    ```
    echo slimarch > /etc/hostname
    ```

Finally reboot

## References

* [Archlinux Installation guide](https://wiki.archlinux.org/index.php/Installation_guide)
* [Archlinux Beginner's guide](https://wiki.archlinux.org/index.php/Beginners\'_guide)
* [Archlinux Intel graphics](https://wiki.archlinux.org/index.php/Intel_graphics)
