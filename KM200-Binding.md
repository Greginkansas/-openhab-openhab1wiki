Documentation of the KM200 binding bundle
**Note:** This Binding will be available starting with version 1.9.0 of the OH1 bindings.

## Introduction

For installation of the binding, please see Wiki page [[Bindings]], or you can add [this JAR](https://openhab.ci.cloudbees.com/job/openHAB1-Addons/lastSuccessfulBuild/artifact/bundles/binding/org.openhab.binding.km200/target/org.openhab.binding.km200-1.9.0-SNAPSHOT.jar) to your `addons` folder.

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
There are two different ways to configure the items.
### Direct access wir defined key:

    DateTime  budDate	"Buderus Date Time[%1$tA, %1$td.%1$tm.%1$tY]" {km200="date_time"}`
    DateTime  budDate	"Buderus Datum  [%1$td.%1$tm.%1$tY %1$tH:%1$tM]" {km200="date_time"}`
    String  budBrand "Brand of the heater [%s]" {km200="sys_brand"}
    String  budType "Type of the heater [%s]" {km200="sys_type"}
    String  budStatus "State of the heater [%s]" {km200="sys_state"}
    String  budFirmware "Firmwareversion [%s]" {km200="ver_firmware"}
    String  budHardware  "Hardwareversion [%s]" {km200="ver_hardware"}

### List of avalible services:

The second way is the definition of user-defined items with direct path to the services. Here you need to know which kind of services is you device supporting. This information you will get from this binding.<BR>
* Set the right configuration to the openhab.cfg file.<BR>
* Now you have to start openhab and take a look to the openhab log file. (/var/log/openhub/openhab.log)<BR>
* Here you will see after 1-2 Minutes after start a table with all supported services and capabilities. It is looking like this:<BR>
* syntax: service:servicepath

##################################################################<BR>
List of avalible services<BR>
readable;writeable;recordable;virtual;type;service;value;allowed;min;max<BR>
1;0;1;0;floatValue;/heatSources/nominalDHWPower;15.0;;;<BR>
1;1;1;0;floatValue;/dhwCircuits/dhw1/setTemperature;60.0;;30.0;80.0<BR>
1;0;0;0;switchProgram;/heatingCircuits/hc1/switchPrograms/Nachmittag;;;<BR>
1;0;1;0;floatValue;/heatingCircuits/hc2/pumpModulation;0.0;;0.0;100.0<BR>
1;0;0;0;stringValue;/heatingCircuits/hc4/status;INACTIVE;INACTIVE|ACTIVE;;<BR>
.....<BR>
##################################################################<BR>
You can copy this table in excel. It is ';' seperated.<BR>
The colums are: <BR>
    readable -> Service is readable, if not then you cannot use it.<BR>
    writable -> It is possible to set values.<BR>
    recordable -> It is possible to create recordings. Not directly supported yet.<BR>
    virtual -> This service is only virtual and not existing of the device. The binding is translating the message to the parent. <BR>
    type -> FloatValue, stringValue, switchProgram (not supported), refEnum ( not relevant), yRecording (not supported). <BR>
    service -> This is the path for the configuration.<BR>
    value -> Value of the service in the time of the init.<BR>
    allowed -> If existing then only this values are possible.<BR>
    min -> The min value for a float.<BR>
    max -> The max value for a float.<BR>

## Switching Programs

With the last commit the binding is now supporting the reading and changing of switching programs.
The communication between the binding and the user is done over virtual services. In the service list are now virtual services included. Every switch program service has now five virtual subservices. They are: <BR>
    weekday -> With this value it'S possible to select a weekday. <BR>
    nbrCycles -> The number of cycles (on+off or day+night) on the selected day. <BR>
    cycle -> With this value it'S possible to select one of the cycle. <BR>
    on/day -> The selected switch time for the on/day type of the selected cycle. <BR>
    off/night -> The selected switch time for the off/night type of the selected cycle. <BR>

    
## Samples

`String  budState "State of the heating [%s]"  {km200="service:/system/healthStatus"}
Number	budTemp  "Temperature heating night [%.1f °C]" {km200="service:/heatingCircuits/hc3/temperatureLevels/night"}`<BR>
For switches you can define which of the allowed values is the one for 'on' and 'off'.<BR>
`Switch  budMode  "Mode [%s]" {km200="service:/heatingCircuits/hc3/operationMode on:auto off:night"}`<BR>
Switching programs:<BR> (you have to select as first the day, second the cycle and then read/write on/day or off/night).
    `String actBudDayHC1 "Day HC1 [%s]" {km200="service:/heatingCircuits/hc1/switchPrograms/Eigen1/weekday" }`<BR>
    `Number nbrBudNbrCyclesHC1 "Cycles HC1 [%d]" {km200="service:/heatingCircuits/hc1/switchPrograms/Eigen1/nbrCycles" }`<BR>
    `Number actBudCycleHC1 "Selected cycle HC1 [%d]" {km200="service:/heatingCircuits/hc1/switchPrograms/Eigen1/cycle" }`<BR>
    `Number actBudPosHC1 "Day  HC1  [%d]" {km200="service:/heatingCircuits/hc1/switchPrograms/Eigen1/day" }`<BR>
    `Number actBudNegHC1"Night HC1  [%d]" {km200="service:/heatingCircuits/hc1/switchPrograms/Eigen1/night" }`<BR><BR>

The supported item types are: Number (for string, float and switching program (cycle, nbrCycles, on/day, off/night), String (for string, float and switching program (weekday)), DateTime (for string and switching program (on/day, off/night) and Switch (for string). <BR>

This binding is automaticly blocking the values to the allowed and limiting them to the min and max capabilities.

