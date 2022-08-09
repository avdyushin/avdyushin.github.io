---
date: 2020-07-04T10:04:46+02:00
title: "Raspberry Pi + Lakka"
tags: ["retro", "raspberry"]
---

## Lakka configuration

1. Connect Ethernet cable or connect to WiFi from GUI.
1. Turn on SSH in Services

### Static IP

Connect via ssh:

```sh
ssh root@lakka.local
```

Edit WiFi settings:

```sh
vi /storage/.cache/connman/wifi_XXXXXX_managed_psk/settings
```

Change IP method to `manual` and set `local_address`:

```ini
[wifi_XXXXXX_managed_psk]
Name=XXXXXXXXXX
SSID=XXXXXXXXXXXXXXXXXXXX
Favorite=true
AutoConnect=true
Passphrase=XXXXXXXX
Frequency=2437
Modified=2020-07-04T15:47:43.045286Z
IPv4.method=manual
IPv4.netmask_prefixlen=24
IPv4.local_address=192.168.0.113
IPv4.gateway=192.168.0.1
IPv6.method=off
IPv6.privacy=disabled
```

Restart WiFi:

```sh
connmanctl disable wifi
connmanctl enable wifi
connmanctl connect wifi_XXXXXX_managed_psk
```

The same for the wired connection (you have to connect cable first to see this config):

```sh
vi /storage/.cache/connman/ethernet_XXXXXX_cable/settings
```

On local machine for easy access can be added: `~/.ssh/config`:

```
Host lakka.wlan
    UserKnownHostsFile ~/.ssh/known_hosts.wlan.lakka
    Hostname 192.168.0.113

Host lakka.eth
    UserKnownHostsFile ~/.ssh/known_hosts.eth.lakka
    Hostname 192.168.0.111
```

Then to connect to ssh via WiFi:

```sh
ssh root@lakka.wlan
```

### Set timezone

```sh
echo "TIMEZONE=Europe/Amsterdam" > /storage/.cache/timezone
```

### Fix HDMI Audio

To edit `config.txt` remount `/flash` with write access:

```sh
mount /flash -o remount,rw
vi /flash/config.txt
```

Add new lines into configuration file:

```ini
# Normal HDMI mode with sound
hdmi_drive=2
hdmi_force_edid_audio=1
```

Use closest to power HDMI port.

### Bluetooth controller

Disable bluetooth ertm to have Xbox one controller paired with bluetooth.


Create file `/etc/modprobe.d/bluetooth.conf` with content:

```
options bluetooth disable\_ertm=Y
```

Verify if it's work after reboot:

```sh
cat /sys/module/bluetooth/parameters/disable_ertm
Y
```

Where `Y` is disabled and `N` enabled.

Pair using `bluetoothctl`.

### Copy BIOSes/ROMs

```sh
scp BIOS/* root@lakka.eth:system/
```

Add missing box-arts as PNG files only.

#### Links

- https://www.lakka.tv
- https://www.lakka.tv/doc/8Bitdo-Wireless-Controller/
- https://www.reddit.com/r/RetroPie/comments/aakkop/xbox_one_s_controller_disable_ertm_persist_on/
