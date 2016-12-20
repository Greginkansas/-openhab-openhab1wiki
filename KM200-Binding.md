_**Note:** This Binding is now in pulling state 

Documentation of the KM200 binding bundle

## Introduction

For installation of the binding, please see Wiki page [[Bindings]], or you can add [this JAR](https://openhab.ci.cloudbees.com/job/openHAB1-Addons/lastSuccessfulBuild/artifact/bundles/binding/org.openhab.binding.ddwrt/target/org.openhab.binding.ddwrt-1.9.0-SNAPSHOT.jar) to your `addons` folder.

The KM200 Binding is communicating with a Buderus Logamatic web KM200 / KM100 / KM50. 
[Description](https://www.buderus.de/de/produkte/catalogue/alle-produkte/7719_gateway-logamatic-web-km200-km100-km50).
It is possible to recive and send parameters like string or float values.

**Important**: If the communication is not working and you see in the logfile errors like "illegal key size" then you have to change the [Java Cryptography Extension to the Unlimited Strength Jurisdiction](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html). 

Adapt your openhab.cfg to your configuration:
* IP address of KN200 to connect to<BR>
    km200:ip4_address=192.168.XXX.XXX<BR>

* There a two ways of de-/encryption password handling:
1.  With the finished private key, here is this one required<BR>
    km200:PrivKey=0000FFFFEEEEDDDDCCCCBBBBAAAA999988887777666655554444333322221111<BR>
2.  --OR-- the binding is creating the key from the md5salt, the device and the private password. Here are all three required
    km200:MD5Salt=111122223333444455556666777788889999aaaabbbbccccddddeeeeffff0000<BR>
    km200:GatewayPassword=AAAABBBBCCCCDDDD<BR>
    km200:PrivatePassword=MYPASSWORD<BR>


## Generic Item Binding Configuration

In order to bind an item to the DD-WRT device, you need to provide configuration settings. The easiest way to do so is to add some binding information in your item file (in the folder `configurations/items`). 

## Switching WIFI

The following items switch WIFI, GUEST_WIFI, and the NAME of the device as string:

    String DEVICE_NAME {ddwrt="routertype"}
    Switch WIFI_24     {ddwrt="wlan24"}
    Switch WIFI_50     {ddwrt="wlan50"}
    Switch WIFI_GUEST  {ddwrt="wlanguest"}

The guest network is usually a virtual network device. There is a bug in the DD-WRT firmware. The activation of this interface needs a workaround so it takes some seconds more as the native devices.

Tested with Archer V2 and DD-WRT v3.0-r30880 std (11/14/16)
