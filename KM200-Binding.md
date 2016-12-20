_**Note:** This Binding is now in pulling state 

Documentation of the KM200 binding bundle

## Introduction

For installation of the binding, please see Wiki page [[Bindings]], or you can add [this JAR](https://openhab.ci.cloudbees.com/job/openHAB1-Addons/lastSuccessfulBuild/artifact/bundles/binding/org.openhab.binding.ddwrt/target/org.openhab.binding.ddwrt-1.9.0-SNAPSHOT.jar) to your `addons` folder.

The KM200 Binding is communicating with a [Buderus Logamatic web KM200 / KM100 / KM50](https://www.buderus.de/de/produkte/catalogue/alle-produkte/7719_gateway-logamatic-web-km200-km100-km50).
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

In order to bind an item to the KM200 device, you need to provide configuration settings. The easiest way to do so is to add some binding information in your item file (in the folder `configurations/items`). 
There are two different ways to configure the items.<BR>
- Direct access wir defined key. In the first version there is only one defined.<BR>
    `DateTime  budDate	  "Buderus Date Time[%1$tA, %1$td.%1$tm.%1$tY]"	{km200="date_time"}`<BR>

- The second way is the definition of user-defined items with direct path to the services. Here you need to know which kind of services is you device supporting. This information you will get from this binding.<BR>
* Set the right configuration to the openhab.cfg file.<BR>
* Now you have to start openhab and take a look to the openhab log file. (/var/log/openhub/openhab.log)<BR>
* Here you will see after 1-2 Minutes after start a table with all supported services and capabilities. It is looking like this:<BR>

## List of avalible services

##################################################################<BR>
List of avalible services<BR>
readable;writeable;recordable;type;service;value;allowed;min;max<BR>
1;0;1;floatValue;/heatSources/nominalDHWPower;15.0;;;<BR>
1;1;1;floatValue;/dhwCircuits/dhw1/setTemperature;60.0;;30.0;80.0<BR>
1;0;0;switchProgram;/heatingCircuits/hc1/switchPrograms/Nachmittag;;;<BR>
1;0;1;floatValue;/heatingCircuits/hc2/pumpModulation;0.0;;0.0;100.0<BR>
1;0;0;stringValue;/heatingCircuits/hc4/status;INACTIVE;INACTIVE|ACTIVE;;<BR>
.....<BR>
##################################################################<BR>
You can copy this table in excel. It is ';' seperated.<BR>
Now you can look what is intresting fo you. Samples:<BR>
`String  budState "State of the heating [%s]"  {km200="service:/system/healthStatus"}
Number	budTemp  "Temperature heating night [%.1f °C]" {km200="service:/heatingCircuits/hc3/temperatureLevels/night"}`<BR>
For switches you can define which of the allowed values is the one for 'on' and 'off'.<BR>
`Switch  budMode  "Mode [%s]" {km200="service:/heatingCircuits/hc3/operationMode on:auto off:night"}`<BR>
The supported item types are: Number (For string and float), String (for string and float), DateTime (for string) and Switch (for string).

