+++
draft = false
date = 2021-04-06T20:55:04+02:00
title = "RetroArch on Raspberry Pi 4"
description = ""
slug = "" 
tags = ["retro", "raspberry"]
categories = []
externalLink = ""
series = []
+++

Requirements:

* Raspberry Pi 4
* SD card with Raspbian
* Another Linux (virtual) desktop to use GParted

Once raspbian image installed it's good to shrink root partition and extract another one for storage.

The goal is to have Raspbian with Desktop and option to boot into RetroArch without using Lakka.
This will give full control over OS like using custom services and applications.

Change raspbian boot order to boot into CLI mode, we will add startup screen later.

### WiFi

Install and configure connman:

```sh
sudo apt install connman
```

Clean old wifi connections (if needed or scan wifi doesn't work):

```sh
sudo mv /etc/wpa_supplicant/wpa_supplicant.conf /etc/wpa_supplicant/wpa_supplicant.conf.orig
sudo reboot
```

Setup connman:

```sh
$ connmanctl
connmanctl> enable wifi
connmanctl> scan wifi
connmanctl> services
WIFI_NAME wifi_XXXXXX_XXX_managed_psk
connmanctl> agent on
connmanctl> connect wifi_XXXXXX_XXX_managed_psk
connmanctl> services
connmanctl> quit
$ ifconfig
```

#### Static IP (Optional)

Edit wifi settings at `/var/lib/connman/wifi_XXXXXX_XXX_managed_psk/settings`

```
[wifi_XXXXXX_XXX_managed_psk]
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

### RetroArch

Get sources:

```sh
curl -LO 'https://github.com/libretro/RetroArch/archive/v1.9.0.tar.gz'
tar -zxvf v1.9.0.tar.gz
cd RetroArch-1.9.0/
```

Install dependencies:

```sh
sudo apt install build-essential git libasound2-dev libavcodec-dev libavdevice-dev libavformat-dev libavresample-dev libdrm-common libdrm-dev libdrm2 libegl1-mesa-dev libfreetype6-dev libgbm-dev libgbm-dev libgbm1 libgles2 libgles2-mesa libgles2-mesa-dev libsdl-image1.2-dev libsdl2-dev libswresample-dev libswscale-dev libudev-dev libv4l-dev libxkbcommon-dev libxml2-dev yasm zlib1g-dev
```

Configure:

```sh
CFLAGS='-march=armv8-a+crc+simd -mcpu=cortex-a72 -mtune=cortex-a72 -mfloat-abi=hard -mfpu=neon-fp-armv8' CXXFLAGS="${CFLAGS}" ./configure  --disable-caca --disable-jack --disable-opengl1 --disable-oss --disable-sdl --disable-sdl2 --disable-videocore --disable-vulkan --disable-wayland --disable-x11 --enable-alsa --enable-egl --enable-floathard --enable-kms --enable-neon --enable-opengles --enable-opengles3 --enable-pulse --enable-udev
```

Build with lakka support:

```sh
make V=1 HAVE_LAKKA=1 HAVE_BLUETOOTH=1 HAVE_NETWORKING=1 -j4
```
This will enable Shutdown menu option, Wi-Fi and Bluetooth options.

```sh
sudo apt install checkinstall
sudo checkinstall --pkgname=retroarch-rpi4 --conflicts=retroarch --pkgversion=1.9.0 --install=no
sudo dpkg -i retroarch-rpi4_1.9.0-1_armhf.deb
```

Run RetroArch to have configuration file to be created.
Set `core_updater_buildbot_cores_url` inside`.config/retroarch/retroarch.cfg`:

```
core_updater_buildbot_cores_url = "http://buildbot.libretro.com/nightly/linux/armv7-neon-hf/latest/"
```

Update Drivers:

```
Video -> glcore
Bluetooth -> bluetoothctl
Wi-Fi -> connmanctl
```

Update assets with online updater and restart retroarch.

### PSP Emulator

```sh
git clone https://github.com/hrydgard/ppsspp
cd ppsspp && git submodule update --init --recursive
CMAKE_ARGS='-DUSING_X11_VULKAN=OFF -DUSE_SYSTEM_FFMPEG=ON' ./b.sh --rpi --libretro
```

And copy core into retroarch:

```sh
cp build/lib/ppsspp_libretro.so ~/.config/retroarch/cores/
```

NOTES: PPSSPP works only with gl (and not glcore) video driver.

### Bluetooth Controller

```sh
$ systemctl status bluetooth
$ sudo rfkill unblock bluetooth # if blocked
$ bluetoothctl
[bluetoothctl]# agent on
[bluetoothctl]# default-agent
[bluetoothctl]# power on
[bluetoothctl]# discoverable on
[bluetoothctl]# pairable on
[bluetoothctl]# scan on
[bluetoothctl]# connect <device_address>
[device_address]# trust <device_address>
```

### Shutdown button

Attach button to pin 9 (GND) and pin 11 (GPIO17) on raspberry board and add to `/boot/config.txt`:

```
dtoverlay=gpio-shutdown,gpio_pin=17
```

Now when you press the button raspberry will shutdown.

### Links
- https://www.reddit.com/r/RetroArch/comments/l158qt/best_performing_retroarch_build_on_a_raspberry_pi/
- https://github.com/hrydgard/ppsspp
- https://pinout.xyz/pinout/
