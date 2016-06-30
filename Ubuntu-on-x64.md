# Introduction
Basically a step by step guide to install [OpenHAB 1.8](http://www.openhab.org) on an Intel x86 machine using a few common bindings.

The content is mostly copy & pasted from other parts of the wiki.

## Operating System

### I've picked the LTS version for stability.

- [Download Ubuntu 14.04 LTS 64bit](http://releases.ubuntu.com/14.04.3/ubuntu-14.04.3-server-amd64.iso)
- Next, make a USB drive to install Ubuntu onto a spare machine:
	- If you're using Windows: [Download Rufus](https://rufus.akeo.ie/downloads/rufus-2.2p.exe)
	- If you're using a unix machine: `sudo ddrescue -f (infile or your .iso file) (outfile or your usb drive)`
		- Example: `sudo ddrescue -f ~/ubuntu-14.04-server.iso /dev/sda1`
		- DDRescue is usually better and more user friendly than just `dd` as it gives you a timer and actual information as it works.

### Install Ubuntu LTS

When going through the installation process, on the Packages options, pick the following options:

- OpenSSH server (To administrate the server using PuTTY or your favorite terminal)
- Samba Fileserver (To get access to the OpenHAB config files from Windows or OS X)

## Dependencies
### Java

We need Java 8, which is not included in Ubuntu 14.04 LTS, so we add a repository and install it:

```bash
sudo apt-add-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

It will ask you to accept the Terms and Conditions of Oracle to use Java.

### MySQL for Persistence

```bash
sudo apt-get install mysql-server
```

Here, it will ask for a root password, saying that "While it is not necessary, it is suggested." Of which it is always a good idea to include a root password. Make sure, of course, to remember this password as well.

Start the Mysql commandline as root

```bash
sudo mysql -u root -p
```

Create a database for OpenHAB

```bash
CREATE DATABASE OpenHAB;
```

Create a user for OpenHAB

```bash
CREATE USER 'openhab'@'localhost' IDENTIFIED BY 'yourpassword';
```

Grant the user permissions for the database

```bash
GRANT ALL PRIVILEGES ON OpenHAB.* TO 'openhab'@'localhost';
```

Quit the Mysql command prompt

```bash
quit
```

### Mosquitto as MQTT broker

MQTT, for those who don't know, is a machine-to-machine (M2M)/"Internet of Things" connectivity protocol.

Mosquito is the program/broker that implements the MQTT protocol.

```bash
sudo apt-add-repository ppa:mosquitto-dev/mosquitto-ppa
sudo apt-get update
sudo apt-get install mosquitto
```

#### Setup TLS on Mosquitto (optional)

Copy the SSL CA to /etc/mosquitto/ca-certificates
copy the SSL certificate and private key (.crt and .key) to /etc/mosquitto/certs

Protect your SSL certificate

```bash
cd /etc/mosquitto/certs
sudo chmod 600 *
sudo chown mosquitto *
```

Configure TLS

```bash
sudo nano /etc/mosquitto/conf.d/tls.conf
```

Add the following

```bash
listener 8883
tls_version tlsv1
cafile /etc/mosquitto/ca-certificate/ca.crt
certfile /etc/mosquitto/certs/server.crt
keyfile /etc/mosquitto/certs/server.key
require_certificate false
```

To restart Mosquitto, try one of the following;

```bash
sudo /etc/init.d/mosquitto restart
sudo service mosquitto restart
```

#### Portforward

Make sure you portforwarded the default mosquitto port 1883 to your server!

## Install OpenHAB

### Configure the repository

```bash
wget -qO - 'https://bintray.com/user/downloadSubjectPublicKey?username=openhab' | sudo apt-key add -
echo "deb http://dl.bintray.com/openhab/apt-repo stable main" | sudo tee /etc/apt/sources.list.d/openhab.list
sudo apt-get update
```

### Install the runtime

```bash
sudo apt-get install openhab-runtime
```

### Get needed addons

```
sudo apt-get install openhab-addon-binding-astro openhab-addon-binding-http openhab-addon-binding-mqtt openhab-addon-binding-mqttitude openhab-addon-binding-networkhealth openhab-addon-binding-wol openhab-addon-binding-zwave openhab-addon-persistence-mysql
```

### Extra addons that are popular

- openhab-addon-binding-hue - [Philips Hue](https://github.com/openhab/openhab/wiki/Hue-Binding)
- openhab-addon-binding-netatmo - [Netatmo Personal Weather Station](https://github.com/openhab/openhab/wiki/Netatmo-Binding)
- openhab-addon-binding-plex - [Plex](https://github.com/openhab/openhab/wiki/Plex-Binding)
- openhab-addon-binding-samsungtv - [Samsung TV](https://github.com/openhab/openhab/wiki/Samsung-TV-Binding)
- openhab-addon-binding-sonos - [Sonos](https://github.com/openhab/openhab/wiki/Sonos-Binding)
- openhab-addon-binding-xbmc - [XBMC](https://github.com/openhab/openhab/wiki/XBMC-Binding)
- openhab-addon-io-myopenhab - [My.OpenHAB](https://github.com/openhab/openhab/wiki/my.openHAB-Persistence
)

## Setup the configuration

### Copy the default configuration

```bash
cd /etc/openhab/configurations/
sudo cp openhab_default.cfg openhab.cfg
```

### openhab.cfg

```bash
sudo nano ./openhab.cfg
```

- Set default persistince to mysql
- Set Mysql server username and password
- Set [MQTT Transport](https://github.com/openhab/openhab/wiki/MQTT-Binding#transport-configuration)
- Set [HTTP Binding cache item for weather](https://github.com/openhab/openhab/wiki/Http-Binding#caching)
- Set Z-Wave port
- Set [Astro binding latitude and longitude](https://github.com/openhab/openhab/wiki/Astro-binding)

### Create items, sitemaps and rules

This is where you should create items, sitemaps and rules.

See the [Configuring the openHAB runtime](https://github.com/openhab/openhab/wiki/Configuring-the-openHAB-runtime) wiki page.

### Symlink com ports for Zwave devices

The [Symlink](https://github.com/openhab/openhab/wiki/symlinks) page has the best instructions on how to add the info for three popular Z-Wave USB Sticks, as well as how to find the needed info for any other USB Sticks.

### Set autostart

```bash
sudo update-rc.d openhab defaults
```

Note that for UBUNTU 15.10 and Raspian Jessie - or if you prefer using `systemctl` - this should be:

```bash
sudo systemctl enable openhab
```

## Share the configuration files using Samba

This is for being able to see and edit the files easily from other machines, if you don't know how to use SSH.

```bash
sudo nano /etc/samba/smb.conf
```

Go to the bottom of the file and add

```bash
[OpenHAB]
comment = OpenHAB Configuration
path = /etc/openhab/configurations
browseable = yes
writeable = yes
guest ok = no
create mask = 0777
directory mask = 0777
```

Restart Samba

```bash
sudo service samba restart
```

Change permissions on the OpenHAB config to 777

```bash
cd /etc/openhab
sudo chmod -R 777 configurations/
```

Now you can go to your Windows machine using `\\hostname\OpenHAB` from your file explorer. Type the string into the Address bar (where the Directory names are at the top) and it should try to access that location. Next, it will ask for your login information, use your Ubuntu credentials.

Now, it should show the files that are available inside of `/etc/openhab/configurations`.

Use Notepad++ since most files have Unix linefeeds.
