This page describes the different places in which the openHAB runtime can be configured and customized:
* [general configuration](Configuring-the-openHAB-runtime#general-configuration)
* [individual configuration](Configuring-the-openHAB-runtime#individual-configuration)  
* [advanced configuration](Configuring-the-openHAB-runtime#advanced-configuration)


**Note**: The configuration files are text files that can be edited with any text editor of your choice. Nevertheless, you may want to use the openHAB designer to edit them, and you will get informed about any syntax error. Note that the expected file encoding is UTF-8.

**Note**: Items and sitemap(s) may be changed during runtime as needed.

**Note**: Use the demo setup if you wish: http://www.openhab.org/downloads.html

## General Configuration - openhab.cfg

The runtime comes with one core configuration file, the file [openhab_default.cfg](https://github.com/openhab/openhab/blob/master/distribution/openhabhome/configurations/openhab_default.cfg).
The purpose of this file is to define all basic settings, such as IP addresses, mail server, folder locations etc.

The file also contains settings for all (optional) bindings. These settings are automatically dispatched to the according binding. For this, all settings come with a namespace (such as "mail:" or "knx:") to identify the associated binding.

First thing after unzipping the runtime should be creating a copy of `openhab_default.cfg` to `openhab.cfg`. All personal settings should always only be done in this copy. This ensures that your settings are not lost, if you unzip a new version of the runtime to your openHAB home directory.

The `openhab_default.cfg` file comes with extensive comments which explain, what settings are available and what can be configured with them. If you have any doubts, please ask on the [discussion group](https://community.openhab.org/).

To activate a binding uncomment the specific settings. 

Example:
The easiest way of configuring a KNX binding is by connecting in ROUTER mode. To do so, enable this: `knx:type=ROUTER` . If you cannot use the ROUTER mode, set it to TUNNEL, but you must then configure the IP: `knx:ip=<IP of the KNX-IP module>`

## Individual Configuration
Besides the *.cfg files and *.xml files (see below: loggin) the configuration folder `${openhab_home}/configurations` consists of dedicated subfolders for specific topics. For each topic, there should be another sub folder, such as `${openhab_home}/configurations/items`.



### Item Definitions
Item files are stored in `${openhab_home}/configurations/items`.

Although items can be dynamically added by item providers (as OSGi services), it is usually very practical to statically define most of the items that should be used in the UI or in automation rules. 

These static definition files follow a certain syntax. This syntax will be explained here. (For the technical interested: This syntax is in fact a [Xtext DSL grammar](https://github.com/openhab/openhab/blob/master/bundles/model/org.openhab.model.item/src/org/openhab/model/Items.xtext).)

Please visit the [Items](Explanation-of-Items) page on how to configure items.

### Sitemap Definitions

Sitemap files are stored in `${openhab_home}/configurations/sitemaps`. 

Sitemaps are a declarative UI definition. With a few lines it is possible to define the structure and the content of your UI screens. 

(For the technical interested: This syntax is in fact a [Xtext DSL grammar](https://github.com/openhab/openhab/blob/master/bundles/model/org.openhab.model.item/src/org/openhab/model/Items.xtext) (Broken Link!).)

Please see page [sitemaps](Explanation-of-Sitemaps) for a description on how to create sitemaps.

### Automation
Rule files are stored in `${openhab_home}/configurations/rules`.

Script files are stored in `${openhab_home}/configurations/scripts`.

Rules provide flexible logic to openHAB for automation which can also use scripts(macros) using related events and actions..

Please visit the [automation](https://github.com/openhab/openhab/wiki/Automation) section for further detailes.

### Persistence
Script files are stored in `${openhab_home}/configurations/persistence`.

Persistences can store item states over a time (a time series).

Please visit the [persistence](https://github.com/openhab/openhab/wiki/Persistence) section for further details. 

### Transformation
Transformations files are stored in `${openhab_home}/configurations/transformation`.

Purpose: tbd...

Please visit the [transformation](https://github.com/openhab/openhab/wiki/Transformation) section for further details.



## Advanced Configuration
### general
You will find the openHAB configuration under /etc/openhab.
If you change configuration files it will not be overwritten by updates or upgrades.

#### ${openhab_home}/etc/default/openhab
    USER_AND_GROUP=openhab:openhab
    HTTP_PORT=8080
    HTTPS_PORT=8443
    TELNET_PORT=5555
    OPENHAB_JAVA=/usr/bin/java

### Logging
#### Configuration



The runtime uses:

##### apt-get installation

`/etc/openhab/logback.xml` and `/etc/openhab/logback_debug.xml`

##### manual installation

`/opt/openhab/configurations/logback.xml` and `/opt/openhab/configurations/logback_debug.xml` 

`logback_debug.xml` is used when you run openHAB in debug mode (start_debug.sh) and can be used as template. 

You can change logback.xml to your needs.

#### Log files

##### apt-get installation

The runtime log: /var/log/openhab/openhab.log
Further log files are placed also into: /var/log/openhab/

##### manual installation

The runtime log: `/opt/openhab/logs/openhab.log`

Further log files are placed also into: `/opt/openhab/logs/`

### Jetty
/etc/openhab/jetty/etc/
### Runtime generated files
/var/lib/openhab
### login.conf
/etc/openhab/login.conf
### users.cfg
/etc/openhab/users.cfg
### quartz.properties
/etc/openhab/quartz.properties