---
title: Setup Samba on Raspberry Pi and FMCB on PS2
date: 2020-04-14T09:59:38+02:00
tags: [retro, raspberry pi]
---

Running PS2 games over network from USB drive connected to Raspberry Pi

1. Setup USB Flash Drive
1. Setup Raspberry Pi
1. Setup FMCB

## USB Storage

Put games you own into `DVD` directory. (Or into `CD` is it's CD)

## Raspberry Pi

### Static IP Address

Use `ifconfig` to discover available network interfaces. Usually `wlan0` is for wireless and `eth0` for ethernet.

Configure in `/etc/dhcpcd.conf`{: .filepath }:

For ethernet set IP address to 192.168.0.111:

```shell
interface eth0
static ip_address=192.168.0.111/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.1 8.8.8.8 fd51:42f8:caae:d92e::1
```

For wireless set IP address to 192.168.0.113:

```shell
interface wlan0
static ip_address=192.168.0.113/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.1 8.8.8.8 fd51:42f8:caae:d92e::1
```

### Mount USB

Discover UUID:

```console
pi@raspberrypi:~ $ ls -l /dev/disk/by-uuid/
total 0
lrwxrwxrwx 1 root root 15 Jun  6 11:28 6341-C9E5 -> ../../mmcblk0p1
lrwxrwxrwx 1 root root 15 Jun  6 11:28 80571af6-21c9-48a0-9df5-cffb60cf79af -> ../../mmcblk0p2
lrwxrwxrwx 1 root root 10 Jun  7 11:46 D7A4-1519 -> ../../sda1
```

USB Stick Drive usually stands for `sdaX` so UUID is `D7A4-1519` in example above.

Mount point:

```console
pi@raspberrypi:~ $ mkdir /media/ps2
pi@raspberrypi:~ $ sudo chown -R pi:pi /media/usb
```

Auto-mount configuration:

```console
pi@raspberrypi:~ $ sudo vi /etc/fstab
```

Add new line and save:

```shell
UUID=D7A4-1519 /media/ps2 vfat auto,nofail,noatime,users,rw,uid=pi,gid=pi 0 0
```

Reload services to re parse `/etc/fstab`{: .filepath }:

```console
pi@raspberrypi:~ $ sudo systemctl daemon-reload
```

And mount drive:

```console
pi@raspberrypi:~ $ mount /media/ps2
```

If drive has been formatted into exFAT file system:

```console
pi@raspberrypi:~ $ sudo apt-get install exfat-fuse exfat-utils
```

And use `exfat` in place of `vfat` in `/etc/fstab`{: .filepath }.

### Samba

Install Samba:

```console
pi@raspberrypi:~ $ sudo apt-get install samba samba-common-bin
```

Configuration:

```console
pi@raspberrypi:~ $ sudo vi /etc/samba/smb.conf
```

```shell
[ps2]
comment = PS2
path = /media/ps2
browseable = yes
writable = yes
only guest = no
create mask = 0777
directory mask = 0777
public = yes
guest ok = yes
```

Restart & verify:

```console
pi@raspberrypi:~ $ sudo /etc/init.d/smbd restart
```

On Mac: Finder -> Go -> Connect to Server... -> smb://192.168.0.113/ps2 -> Connect as Guest

### FMCB

OPL -> Network setup:

```
- PS2 -
IP address type Static
IP address      192.168.0.112
Mask            255.255.255.0
Gateway         192.168.0.1
DNS Server      192.168.0.1

- SMB Server -
Address type    IP
Address         192.168.0.113 // Raspberry Pi IP
Port            445

Share           ps2           // Raspberry Pi Samba Shared Directory
User            guest
```

Don't forget to save configuration.

## Links

- [How to mount a USB flash disk on the Raspberry Pi](https://www.raspberrypi-spy.co.uk/2014/05/how-to-mount-a-usb-flash-disk-on-the-raspberry-pi/)
- [Tree Structue](https://bitbucket.org/ShaolinAssassin/open-ps2-loader-0.9.3-documentation-project/wiki/tree-structure)

