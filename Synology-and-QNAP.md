
## Synology Diskstation

A package of openHAB 1.8.1 and 2.0.x for [Synology Diskstations](http://www.synology.com/en-us/products/index) is stored available on [github](https://github.com/cniweb/openhab-syno-spk/releases)

[Synology discussions on the openHAB Community](https://community.openhab.org/t/synology-diskstation/1446)

**Quick Install OpenHAB V2 until it's in the official docs:**

Current releases of OpenHAB2 require Oracle Java due to the generation of certificates based on [EC](https://en.wikipedia.org/wiki/Elliptic_curve_cryptography) which is not available in free Java implementations. If you see the following error you are likely using OpenJDK:
    java.security.NoSuchAlgorithmException: EC KeyPairGenerator not available

The easiest way to install Java is to include the [PC-Loadletter Repo](http://packages.pcloadletter.co.uk/). Download [Java8 SE](http://www.oracle.com/technetwork/java/embedded/index.html) [(latest ARMv6)](http://www.oracle.com/technetwork/java/embedded/embedded-se/downloads/javase-embedded-downloads-2209751.html) [(Java Cryptography Extension if required)](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html) and put them in the public folder of your NAS. Then start the installation of Java8 SE.

Now download and install the [github](https://github.com/cniweb/openhab-syno-spk/releases) package.

**Deprecated builds and older information can be found here:**

A package of OpenHAB 1.5.0 for [Synology Diskstations](http://www.synology.com/en-us/products/index) is stored at [OpenHAB google groups](https://groups.google.com/d/msg/openhab/lrzcZDYI3Ug/hLJF-sUUjgMJ) and on the package server [https://www.hofrichter.at/sspks](https://www.hofrichter.at/sspks/index.php?fulllist=true).

## Synology Information below is for older versions

This packages can be installed in DSM via the package manager -> manual installation or by adding https://www.hofrichter.at/sspks/ as package source - there is a [tutorial on the Synology support pages](http://www.synology.com/en-us/support/tutorials/500) about how to do that.
This package is tested on DS213+ with oracle's java 7 from [PC load letter](http://pcloadletter.co.uk/2011/08/23/java-package-for-synology/).  
In the Synology package manager you can
* install
* start
* stop
* uninstall

OpenHAB.  
This package comes without any demo files - so the UI will not show much without giving OpenHAB some configurations ;-)
It takes some time after the start of OpenHAB before the UI starts responding (less than a minute on a DS213+).

####Paths
OpenHAB is installed at `/var/packages/OpenHAB/target/` (which is linked to `/volume1/@appstore/OpenHAB`).  
If the directory `/volume1/public/OpenHAB/configurations` exists, this directory will be used for all OpenHAB configuration files. This should make it easier to work with the OpenHAB designer installed on your PC.

If the directory `/volume1/public/OpenHAB/addons` exists, this directory will be used for all OpenHAB addons (bindings) files.

####Ports
The UI is on port 8081, the console (via telnet) is on port 5555.  

####Demo files
In the [OpenHAB google groups](https://groups.google.com/d/msg/openhab/lrzcZDYI3Ug/94XD81A9TYAJ) and on the package server https://www.hofrichter.at/sspks is an install-able package of the OpenHAB 1.4.0 demo files (only the demo files!) for Synology NAS. Prerequisite is that the OpenHAB package is installed.

After the installation of OpenHAB and the demo files http://your.synology.URL:8081/openhab.app?sitemap=demo should show the UI with the demo configuration.

"Text to speech" is disabled in configurations/rules/demo.rules otherwise the demo files are unchanged. This change was necessary (at leased on a DS213+) else OpenHAB stops responding.
Give OpenHAB a minute or so after the start of OpenHAB before you expect any response in the UI ;-)
####Kernel drivers
Some bindings (e.g. [EnOcean](https://github.com/openhab/openhab/wiki/EnOcean-Binding)) work with USB-sticks that require some kernel drivers. For qoric CPU Diskstations (e.g. DS213+) there is a short guide how to install those drivers at [Samples-Tricks](https://github.com/openhab/openhab/wiki/Samples-Tricks#enocean-binding-on-synology-ds213-kernel-driver-package).  

## QNAP Turbo Station

QPKG for QNAP on GitHub can be found here -> https://github.com/openhab/openhab-qnap-qpkg

There is a QPGK package to install openHAB v1.7 on a QNAP Turbo Station NAS.
http://forum.qnap.com/viewtopic.php?f=320&t=95315&p=423268