# Content

* [Installation via **apt-get**](#apt-get)
* [**Manual installation** (alternatively - including openHAB Designer)](#manual-installation-alternative-approach)
* [Yocto Layer](#yocto-layer)
* [Via Chef](#via-chef)



**Note**: for Hardware specific details for Synology Diskstation, QNAP, and ARM boards like Raspberry Pi  visit:
* [ARM based systems](https://github.com/openhab/openhab/wiki/Hardware-FAQ)
* [Synology and QNAP servers] (https://github.com/openhab/openhab/wiki/Synology-and-QNAP)

# Overview
openHAB is a Java application and is expected to run on all platforms where JVM 1.8 or later is available. This includes Windows, Mac OS X, and Linux on x86, x86_64, and ARM architectures.  openHAB can be run on laptops, desktop computers, or  ARM based single-board computers.

# apt-get
* Note: As of the release of openHAB 1.7.1 on 26. Juli 2015 the deb-repository is digitally signed! Therefore you will have to add the "openHAB Bintray Repositories" gpg key to your apt keyring if you want to use the repo. This is also necessary for the installation of releases prior to 1.7.1.. The key installation is described below.

## Installation
1. Install Java if 1.8 or higher is not already installed.  The following will display your current Java version.

  ```
  java -version
  ```
1. Add the openHAB Bintray Repositories key to the apt-keyring

  using wget:
  ```
  wget -qO - 'https://bintray.com/user/downloadSubjectPublicKey?username=openhab' | sudo apt-key add -
  ```
  or using curl:
  ```
  curl 'https://bintray.com/user/downloadSubjectPublicKey?username=openhab' | sudo apt-key add -
  ```
1. Add openHAB apt repository to the apt sources list (Note: the current openhab.list file will be overwritten)

  ```
  echo "deb http://dl.bintray.com/openhab/apt-repo stable main" | sudo tee /etc/apt/sources.list.d/openhab.list
  ```
1. Resynchronize the package index

  ```
  sudo apt-get update
  ```
1. Install the openHAB runtime

  ```
  sudo apt-get install openhab-runtime
  ```
1. If you have more than one OpenHAB binding with a USB device (Z-Wave, RFXCOM, etc), refer to [[symlinks]]

1. Start openHAB - manually
  1. Init based on sysVinit (e.g. Debian 7 / Ubuntu 14.x and earlier)

  ```
  sudo /etc/init.d/openhab start
  sudo /etc/init.d/openhab status
  ```
  2. Init based on systemd (e.g. Debian 8 / Ubuntu 15.x and higher)

  ```
  sudo systemctl start openhab.service
  sudo systemctl status openhab.service
  ```
1. Start openHAB - at system startup
  1. Init based on sysVinit (e.g. Debian 7 / Ubuntu 14.x and earlier)

  ```
  sudo update-rc.d openhab defaults
  ```
 or using Init based on systemd (e.g. Debian 8 / Ubuntu 15.x and higher)

  ```
  sudo systemctl daemon-reload
  sudo systemctl enable openhab
  ```
1. Install the add-ons / bindings as you need them (see list on right side-bar)

  ```
  sudo apt-get install openhab-addon-${addon-type}-${addon-name}
  ```
  Examples:
  ```
  sudo apt-get install openhab-addon-binding-knx
  sudo apt-get install openhab-addon-persistence-rrd4j
  sudo apt-get install openhab-addon-io-dropbox
  sudo apt-get install openhab-addon-action-twitter
  ```
  A list of all available packages can be retrieved with

  ```
  sudo apt-cache search openhab
  ```
1. Configure your system as outlined here: [configuration](https://github.com/openhab/openhab/wiki/Configuring-the-openHAB-runtime).

1. If you wish to use a USB zwave stick or other USB/serial device you will need to add the "openhab" users to the "dialout" group:

  ```
  sudo usermod -a -G dialout openhab
  ```

  and then reboot.

1. Test it

Point your browser to `http://localhost:8080/openhab.app?sitemap=yourname` and you should see your sitemap.

## Performance Tuning
Once your system is stable and configured the way you want it, review [[Performance Tuning|Performance]]



## To Upgrade
Note: changed configuration files will be retained even on upgrades.

    sudo apt-get update
    sudo apt-get upgrade

## File locations
* **service configuration** /etc/default/openhab
* **site configuration**  /etc/openhab
* **log files**  /var/log/openhab
* **userdata like rrd4j databases**  /var/lib/openhab
* **openHAB engine, addons and /webapps folder**  /usr/share/openhab


## Advanced Setup
### Apt Repository Distributions
1. stable

  The latest stable openHAB release will be installed.
1. testing

  The latest release candidate of openHAB will be installed.
1. unstable (not yet available)

  The latest snapshot release of openHAB will be installed.
1. version based distribution names (1.7.0, ...)

  All Releases are available as with a distribution name corresponding to the version name.
If you want to stick your installation to a specific version use one of these distribution names.
The installation will only be upgraded if you change the sources.list to another version.
Therefore "apt-get update && apt-get upgrade" can be safely used for the other linux software.

  Examples:
  ```
  echo "deb https://dl.bintray.com/openhab/apt-repo 1.7.0 main" | sudo tee -a /etc/apt/sources.list
  echo "deb https://dl.bintray.com/openhab/apt-repo 1.7.0.RC1 main" | sudo tee -a /etc/apt/sources.list
  ```

## Snapshot builds
Not yet available.

### [Java 8 Installation for Ubuntu or Debian](http://www.webupd8.org/2014/03/how-to-install-oracle-java-8-in-debian.html)

```
su -
echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | tee /etc/apt/sources.list.d/webupd8team-java.list
echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | tee -a /etc/apt/sources.list.d/webupd8team-java.list
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886
apt-get update
apt-get purge openjdk* # (to remove previously installed lower versions)
apt-get install oracle-java8-installer
exit
```

# Manual installation (alternative approach)
**WARNING**: this quick setup is an example for a KNX environment. You need different addons and configurations for other bindings.


## Installing the openHAB runtime

The openHAB runtime comes as a platform-independent zip file.
To install it, follow these simple steps:

1. You will need to install Java if not already installed. Go to http://java.com/ to get it. For ARM based systems and Synology Diskstation, see [Hardware FAQ](https://github.com/openhab/openhab/wiki/Hardware-FAQ) for instructions on getting Java.
1. Unzip the `openhab-runtime-<version>.zip` to where it is intended to be running from, e.g. `/opt/openhab`.  
1. Copy the bindings you have downloaded -`knx-binding-<version>.jar` and `http-binding-<version>.jar`- to the "addons" directory.
1. Create a personal configuration file `configurations/openhab.cfg` and add the appropriate configuration parameters from `configurations/openhab_default.cfg` (depending on the bindings you've copied).

### OPTIONAL: Installing the openHAB designer

The openHAB designer comes as a platform-*dependent* zip, so choose the right type for your platform.
To install it, follow these simple steps:

1. Unzip the `openhab-designer-<platform>-<version>.zip` to some directory, e.g. `/opt/openhab-designer`
1. Launch it by the executable `openHAB-Designer`
1. Select the "configurations" folder of your runtime installation in the folder dialog that is shown when selecting the "open folder" toolbar icon.

## Configuring the server
For please visit the [configuration](https://github.com/openhab/openhab/wiki/Configuring-the-openHAB-runtime) page(s).


## Start the server!

1. Launch the runtime by executing the script `start.sh`

## Go test it!

openHAB comes with a built-in user interface. It works on all webkit-based browsers like Chrome, Safari, etc. Point your browser to `http://localhost:8080/openhab.app?sitemap=yourname` and you should be looking at your sitemap.

# Yocto Layer
For people who make their own linux distribution using Yocto, there is a openHAB layer available:

https://github.com/ulfwin/meta-openhab

You can also find it through the very convenient layer search site:

http://layers.openembedded.org/layerindex/branch/master/layers/

The layer contains one recipe that install both the runtime engine and addons. Check the README for specific instructions for the layer. How to add and use a layer is not covered here (you need to know that in order use Yocto in the first place), but some explanation can be found here:

http://www.yoctoproject.org/docs/1.6/dev-manual/dev-manual.html#understanding-and-creating-layers

# Via Chef

There is a chef cookbook available at [github.com/JustinAiken/openhab-cookbook](https://github.com/JustinAiken/openhab-cookbook).  It can install from the Debian packages or via source.