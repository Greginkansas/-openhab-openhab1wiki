Documentation of the CometVisu Backend

## Introduction

This adds a backend for the web based visualization CometVisu (http://www.cometvisu.org). The CometVisu is a highly customizable visualization, that runs in any browser. Despite the existing browser based UI´s in openHAB, the CometVisu does not read the sitemaps. The layout is defined with an XML-based configuration file.

This is just a short overview, more details will be added soon!

## Requirements

* openHAB 1.4 or greater<br>
Note: The latest version can be downloaded from [here](https://github.com/openhab/openhab/releases)
* CometVisu 0.8.0 or greater (https://github.com/CometVisu/CometVisu/releases).<br>
It might be useful (but not recommended!) to work with the latest Git version of CometVisu as well.<br>
Please note that this is the development version of the CometVisu, which might be broken. It is not recommended to use this unless you know what you are doing!<br>
On a Raspberry Pi, the installation of the Git version works like this:<br>
 1. Installation of subversion on the RasPi:<br>
`sudo apt-get update` <br>
`sudo apt-get install subversion`<br>
 2. Download of the CometVisu Git<br>
Assumptions: openHAB is installed in the directory openHAB, CometVisu is located in webapps/cometVISU<br>
`cd openHAB/webapps`<br>
`svn co https://github.com/CometVisu/CometVisu/trunk/src/ cometVISU`<br>
 3. For an update later on you just need to do this<br>
`cd openHAB/webapps`<br> 
`svn update cometVISU`<br>
Note: This information ist taken from [here](http://www.cometvisu.de/wiki/index.php?title=CometVisu/HowTo_install_the_development_version_on_the_WireGate). As long as the PHP runtime is not available (see Known Problems), the chmod described there is not required.

## Installation

* Copy the addon org.openhab.io.cv*.jar from the [addon packet](http://www.openhab.org/downloads.html) to the openHAB addon folder
* Then just extract the "release" folder (the one which contains the index.html file) of the [downloaded CometVisu archive](https://github.com/CometVisu/CometVisu/releases) in openHAB's webapps folder and rename it to, e.g. cometVISU.

## Configuration
If you don´t use the given example below, please make sure that you correctly configure openHAB as backend in the CometVisu-Config by adding `backend="oh"` to the root pages-element.
`<pages backend="oh" lib_version="8" design="metal" screensave_page="Übersicht" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../visu_config.xsd">
`

[This](https://www.dropbox.com/s/5ip5fv5h5d4st9v/cometVISU_openHAB.zip) is a first small sample implementation. Actually only the items below are used in the example config:
* number
* switch
* contact
* dimmer

Please feel free to add more items. Other possible item types are:
* rollershutter
* string
* color

Some basic examples:
* ColorItem (supported since CometVisu-Release 0.8.2) => 
`<colorchooser>
  <label>Color</label>
  <address transform="OH:color" variant="rgb">ITEM_NAME</address>
</colorchooser>`

Please note: You have to add the colorchooser plugin in the meta>plugins section of you config

## Known problems

Some parts of the CometVisu need a PHP runtime. As this is not included in Jetty, the following parts do not work:
* Editor
* Configuration check
* Configuration upgrade (only needed if you upgrade your CometVisu to a new release)

Currently the only workaround is to build, edit and maintain your CometVisu configuration manually with an XML editor of your choice.

### 403 error 
If you get an 403 - Access Denied error, when you try to open the cometVISU in your browser you have not copied the correct release folder into the webapps/cometVISU/ folder. Please check if there is a subfolder with the exact name "release/", which contains an index.html file and copy the content of this folder in your webapps/cometVISU/ folder.

## Screenshots

Screenshots are available here: [http://www.cometvisu.org/wiki/CometVisu](http://www.cometvisu.org/wiki/CometVisu)

## Links

* German CometVisu Support Forum: http://knx-user-forum.de/forum/supportforen/cometvisu
* User documentation for the CometVisu: http://www.cometvisu.org/
* GitHub project page of the CometVisu: https://github.com/CometVisu/CometVisu