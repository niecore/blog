---
title: "Use a cheap ublox gps dongle for astroberry time and coordindates"
date: 2022-04-21T10:13:04+01:00
draft: false
tags: 
  - astrophotography
  - raspberrypi
---

One disadvantage of running astroberry on a raspberry pi is that you dont have a built in real time clock and are asked to setup the current time every time you restart the device unless you are connected to the internet.
Therefore I bought a cheap usb gps dongle that I want to connect to my astrophotography rig which can retrieve the current time from the gps satellites. That will safe me time during the setup phase of the astrophotography rig.

{{< imgresize "img/u-blox7.PNG" "400x400" "U-Blox Device on Amazon" >}}

I atteched the device to my astroberry and checked if it is detected correctly (see Ublox with iwth id U-Blox AG [u-blox 7]):

```
astroberry@astroberry:~ $ lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 006: ID 067b:2303 Prolific Technology, Inc. PL2303 Serial Port
Bus 001 Device 005: ID 1546:01a7 U-Blox AG [u-blox 7]
Bus 001 Device 004: ID 04a9:323b Canon, Inc. EOS Rebel T4i
Bus 001 Device 003: ID 03c3:120c
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

We want to use the indi driver `indi_gpsd` which makes use of the `gpsd` running on the raspberry pi. In my version of astroberry the `gpsd` was already installed and running.
If this is not the case for you run the following commands:

```
sudo apt install gpsd
sudo systemctl start gpsd
sudo systemctl enable gpsd
```

After that I disabled the virtual gps of astroberry:

```
astroberry@astroberry:~ $ sudo systemctl stop virtualgps.service
astroberry@astroberry:~ $ sudo systemctl disable virtualgps.service
Removed /etc/systemd/system/multi-user.target.wants/virtualgps.service
```

With help of the utility gpsmon I was then able to check the status of the `gpsd`:

{{< imgresize "img/gpsmon.PNG" "400x400" "Screenshot of the gpsmon output" >}}

 Warning! the gps dongles antenna is very bad and the gps was not connecting in my case even though I tested it next to a window. Only when the green led of the gps dongle is blinking green you have a gps satellite fix.

Finally you can add the driver `indi_gpsd` to your equipment list and start ekos:

{{< imgresize "img/indi_gpsd.PNG" "400x400" "Screenshot of the ekos indi_gpsd driver" >}}