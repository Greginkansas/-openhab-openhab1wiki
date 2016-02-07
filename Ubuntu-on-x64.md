##Introduction
Basically a step by step guide to install [OpenHAB 1.8](http://www.openhab.org) on an Intel x86 machine using a few common bindings. 
The content is mostly copy & paste from other parts of the wiki.

## Operating System
I've picked the LTS version for stability.

- Download Ubuntu 14.04 LTS 64bit: http://releases.ubuntu.com/14.04.3/ubuntu-14.04.3-server-amd64.iso
- Download Rufus: https://rufus.akeo.ie/downloads/rufus-2.2p.exe
- Create bootable USB using Rufus and the ISO file.

Install Ubuntu LTS

Pick the following options:
- OpenSSH server (To administrate the server using PuTTY)
- Samba Fileserver (To get access to the OpenHAB config files from Windows)

## Dependencies
###Java
We need Java 8, which is not included in Ubuntu 14.04 LTS, so we add a repository and install it.
```bash
sudo apt-add-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

###MySQL for persistence
```bash
sudo apt-get install mysql-server
```

Start the Mysql commandline as root
```bash
mysql -u root -p
```

Create a database for OpenHAB
```
CREATE DATABASE OpenHAB;
```
Create a user for OpenHAB
```
CREATE USER 'openhab'@'localhost' IDENTIFIED BY 'yourpassword';
```
Grant the user permissions for the database
```
GRANT ALL PRIVILEGES ON OpenHAB.* TO 'openhab'@'localhost';
```
Quit the Mysql command prompt
```
quit
```

###Mosquitto as MQTT broker
```bash
sudo apt-add-repository ppa:mosquitto-dev/mosquitto-ppa
sudo apt-get update
sudo apt-get install mosquitto
```

####Setup TLS on Mosquitto (optional)
copy the SSL CA to /etc/mosquitto/ca-certificates
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
```
listener 8883
tls_version tlsv1
cafile /etc/mosquitto/ca-certificate/ca.crt
certfile /etc/mosquitto/certs/server.crt
keyfile /etc/mosquitto/certs/server.key
require_certificate false
```

Restart Mosquitto, try one of the following; 
* /etc/init.d/mosquitto restart
* service mosquitto restart

####Portforward
Make sure you portforwarded the default mosquitto port 8883 (or 1883) to your server!  

## Install OpenHAB 
###Configure the repository
```bash
wget -qO - 'https://bintray.com/user/downloadSubjectPublicKey?username=openhab' | sudo apt-key add -
echo "deb http://dl.bintray.com/openhab/apt-repo stable main" | sudo tee /etc/apt/sources.list.d/openhab.list
sudo apt-get update
```

###Install the runtime
```bash
sudo apt-get install openhab-runtime
```

## Get needed addons
```bash
sudo apt-get install openhab-addon-binding-astro
sudo apt-get install openhab-addon-binding-dsmr
sudo apt-get install openhab-addon-binding-http
sudo apt-get install openhab-addon-binding-hue
sudo apt-get install openhab-addon-binding-mqtt
sudo apt-get install openhab-addon-binding-mqttitude
sudo apt-get install openhab-addon-binding-netatmo
sudo apt-get install openhab-addon-binding-networkhealth
sudo apt-get install openhab-addon-binding-plex
sudo apt-get install openhab-addon-binding-samsungtv
sudo apt-get install openhab-addon-binding-sonos
sudo apt-get install openhab-addon-binding-wol
sudo apt-get install openhab-addon-binding-xbmc
sudo apt-get install openhab-addon-binding-zwave
sudo apt-get install openhab-addon-io-myopenhab
sudo apt-get install openhab-addon-persistence-mysql
```

## Get the correct configuration
```bash
cd /etc/openhab/configurations/
sudo cp openhab_default.cfg openhab.cfg
```

### openhab.cfg
- Set security option external, for my.openhab
- Set default persistince to mysql
- Set Mysql server, username and password
- Set MQTT Transport
- Set HTTP Binding cache item for weather
- Set Philips Hue ip and secret
- Set Sonos UDN
- Set Samsung TV host
- Set Z-Wave port
- Set Netatmo clientid, clientsecret and refreshtoken
- Set Astro binding latitude and longitude 
- Set Plex host
- Set DSMR port and gas channel

### Create items, sitemaps and rules
This is where you should create items, sitemaps and rules.

###symlink com ports for Zwave and DSMR devices
```bash
sudo nano /etc/udev/rules.d/50-usb-serial.rules
```
And add for the Aeotec Z-Stick II
```
SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{product}=="CP2102 USB to UART Bridge Controller", SYMLINK+="usb_zwave", GROUP="dialout", MODE="0666"
```
or for the Aeotec Gen5 USB stick:
```
SUBSYSTEM=="tty", ATTRS{idVendor}=="0658", ATTRS{idProduct}=="0200", SYMLINK+="usb_zwave", GROUP="dialout", MODE="0666"
```

And for the DSMR cable
```
SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{product}=="FT232R USB UART", SYMLINK+="usb_dsmr", GROUP="dialout", MODE="0666"
```
Add the symlinks to the Java args
```bash
sudo nano /etc/default/openhab
```
Look in the file for JAVA_ARGS and add the following
```
-Dgnu.io.rxtx.SerialPorts=/dev/usb_dsmr:/dev/usb_zwave
```

###Set autostart
```bash
sudo update-rc.d openhab defaults
```

##Share the configuration using Samba
```bash
sudo nano /etc/samba/smb.conf
```
Go to the bottom of the file and add
```
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
sudo /etc/init.d/samba restart
```
Change permissions on the OpenHAB config to 777
```bash
cd /etc/openhab
sudo chmod -R 777 configurations/
```

Now you can go to your machine using \\hostname\OpenHAB from your Windows machine (use your Ubuntu credentials) and edit the configuration files.
Use Notepad++ since most files have Unix linefeeds