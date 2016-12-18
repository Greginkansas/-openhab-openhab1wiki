
## Introduction

This page summarizes how to get openHAB server running on specific ARM hardware.

## ARM Single-Board Computers

ARM-based computers are a popular choice due to low cost, small size, low power consumption and fan-less operation. They are usually extensible with available add-on components and via programmable GPIO.

**ONLY use the vendor forums recommended Wifi, SD card and power supply.**  These systems have little headroom in  their specs and may be flaky with other devices.  A lot of power supplies don't actually deliver what they claim, so again, go with a recommended model or you will get random flakiness and reboots.  If you experience dropped or duplicated packets from a USB device (keyboard, wifi, Z-Wave), search the vendor's forums for "keyboard not working".  You may need to upgrade firmware or use specific ports and settings for the USB devices.


| System   | Cores | Ghz | Soc/CPU | Gb RAM | IO  | Storage | Primary OS  |
| -------- | ------|-----|-------| ------ | --- |---|---|
| RaspPi 1/B | 1     | 0.7 | A5    | 0.5    | GPIO | SD |Raspbian |
| RaspPi 2  | 4     | 0.9 | A7    | 1.0    | GPIO | SD |Raspbian |
| RaspPi 3  | 2     | 1.0 | A7    | 1.0    | GPIO | SD |Raspbian |
| BeagleBlk | 2     | 1.0 | A8   | 0.5    | GPIO | SD | Ubuntu |
| BananaPi  | 2     | 1.0 | A20/A7 | 1.0    | GPIO | µSD SATA |Bananian |
| BananaPi M3 | 8    | 1.2-1.8 | A83T  | 2.0 | GPIO | µSD SATA eMMC |Bananian |
| BananaPi M64 64Bit | 4   | 1.2 | A53    | 2.0    | GPIO | eMMC |Bananian |
| ODROID C1+ | 4     | 1.5 | A5  | 1.0    | GPIO | µSD, eMMC Slot| Ubuntu |
| ODROID C2 64Bit| 4     | 1.5 | A53 | 2.0    | GPIO | µSD, eMMC Slot |Ubuntu |

### Raspberry Pi2

![Raspberry PI 2](https://pbs.twimg.com/media/B82celWIIAAx90U.jpg:large)

* Complete compatibility with Raspberry Pi 1

 A proven choice for small residential installations. 

**NOTE** There have been issues with the zwave binding on the RPi. Refer to the [[z-wave binding]] for more information


#### Installation

1. Install Raspbian Wheezy release 2014-01-07 or later. It includes Java 7 JRE that meets openHAB  JVM requirements.

1. Install openHAB -  - as described in
[[Linux-and-OS-X]]

Optionally, a complete JDK 7 or 8 for Linux ARM v6/v7 Hard Float ABI is available for download from Oracle or from [Apt Repository](https://github.com/openhab/openhab/wiki/Apt-Repository).

#### Tweaking Performance

Stay up to date with rpi-update (https://github.com/Hexxeh/rpi-update/)

##### tmpfs on SD card based devices
[This article](http://knx-user-forum.de/forum/supportforen/openhab/25012-kontinuierliche-schreibaktivit%C3%A4ten-openhab) has tracked down the reason for the high I/O load you may experience when running openHAB on devices which use a SD card as storage media for the root (and /tmp) filesystem.
Because the forum article is in german, here is a short summary:

* The Apache Fileinstall Bundle is used to continuously check the _$OPENHAB_HOME/addons_ folder in openHAB
* The bundle writes nearly continously to _/tmp/fileinstall-..._
* When uncommenting/setting **RAMTMP=yes** in _/etc/default/tmpfs_ these I/O operations will be RAM access rather than slow SD card I/O
* (on Raspbian Jessie, RAMTMP is ignored - set this with "systemctl enable tmp.mount")

##### Change GPU memory usage
For headless, reduce memory down to 16, this can be done by using **raspi-config**

##### Disabling TV Service
For headless, add these to /etc/rc.local:

    # Limit GPU IRQs
    fbset -xres 16 -yres 16 -vyres 16 -depth 8
    /opt/vc/bin/tvservice -o

Overclocking does not seem to have a significant impact.



### CubieBoard, ODroid, BeagleBone Black

These systems work well with OpenHAB for relatively large installations.

**1. Install Linux distro**

Recommended Linux distros:

* **ODroid C1**:  Ubuntu 14.04 

* **BeagleBone Black**: Ubuntu 14.04 

* **CubieBoard2 or CubieTruck**  - Use [Cubiuntu](http://cubiuntu.com)

**2. Install JAVA** - see [java installation](http://www.webupd8.org/2012/09/install-oracle-java-8-in-ubuntu-via-ppa.html)

**3. Install and setup OpenHAB**   - as described in
[[Linux---OS-X]]




### Banana Pi (bPi) 

Recommended distro: raspbian.

 All about performance tweaking from Rpi above is applicable for the bPi too, but the most important thing is the SATA Port which enables users to work around the poor I/O performance of the SD card subsystem.

The SATA connection is the key if you want to run an openhab server (also a not-so-small installation) on such board. You can switch from SD card to SATA disks like described [here](http://www.htpcguides.com/move-linux-banana-pi-sata-setup/).

#### Performance impact
The following data is taken from the monitoring of my openhab server which has about 75 items, 50 rules and 80 sitemap-items.

Without changing anything else but the mass storage from SD card (Transcend SDHC UHC-I X600) to SSD (Crucial M500) overall cpu load dropped from 2.x to about 0.8.

![CPU load](https://lh5.googleusercontent.com/LbTyD62I1Nk3jT-sa3BJ48PQiF9xxjDHyJUVuLf4gmbcX6A85fxKy-kMGLsTd1Dm7j-4AGB_atU=w1790-h805)

Key to this was the reduction of I/O wait (writing to SD card) from 25% to below 5% in cpu utilization. 

![CPU utilization](https://lh3.googleusercontent.com/iDnCNJpI8jFoCYTSWzq2mHPg4DySw8D9FWxfwedqMK5DT5Lhdp0VjWQq1V6296IKOZzc2P6Da5E=w1790-h805)

So far the combination of Banana Pi and SATA SSD can be highly recommended. 


#### Power supply of SATA devices

Be aware of power issues if you plan to use a normal (mechanical) hard disk. The bPi is only able to supply the 5V part of SATA power. Mechanical 3.5'' hard disks usually utilize 12V for the mechanical parts (disk spin motor, head control) and 5V for the electronical part. SATA SSD's and most 2.5'' HD's only use 5V and in general they do not require the 12V supply power to be present. For running an bPi with SATA SSD or 2.5'' HD the bPi-specific SATA/Power cable is sufficient in most cases.