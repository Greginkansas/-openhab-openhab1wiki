AKM868 Binding WIKI

since 1.8.0

# Introduction
This binding is for users coming from the proprietary homeautomation-system "IP-Symcon". If you have bought some hardware from them, you can use this binding to enable your AKM868 presence detection system. The following hardware was used to do presence detection with the AKM-868 controller and a OVO868-tracker for your key-ring:

![AKM-Module](/openhab/openhab/blob/master/bundles/binding/org.openhab.binding.akm868/src/main/resources/AKM868.png) ![LAN-T Module](/openhab/openhab/blob/master/bundles/binding/org.openhab.binding.akm868/src/main/resources/LAN-T868.png)
![868xOVO Tracker](/openhab/openhab/blob/master/bundles/binding/org.openhab.binding.akm868/src/main/resources/Tracker-868xOVO.png)

For installation of the binding, please see Wiki page [[Bindings]].

# Binding configuration

First of all you need to introduce your LAN-T868 adapter in the openhab.cfg file (in the folder '${openhab_home}/configurations').

    ################################ AKM-868 Binding ##########################################    
    # 
    # host: the IP of the adapter LAN-T 868
    akm868:host=192.168.1.13
    # port: the port of the adapter LAN-T 868 (default is 10001)
    akm868:port=10001
    # timeout: time in milliseconds. If the AKM-Controller does not send an update within that
    # timeframe, the switch item is changing to OFF
    akm868:timeout=120000

# Item Binding Configuration

In order to bind an item to the device, you need to provide configuration settings. The easiest way to do so is to add some binding information in your item file (in the folder configurations/items`). The syntax of the binding configuration strings accepted is the following:

    akm868="id=x,channel=y"

The **id** represents the ID of your OVO-Tracker.

The **channel** can be one of the following values:
- 0 = notify on pings from the tracker
- 1 = when the button of the tracker was pressed
- 5 = when the button of the tracker was pressed longer


A sample configuration could look like:

This item would turn the Switch to **ON** as soon as openHAB receives a PING from the tracker. It will turn the Switch to **OFF** if the tracker doesn't send another PING within the given timeframe (**akm868:timeout** value from the **openhab.cfg**):

    Switch PresenceMichael "Key Michael" <present> {akm868="id=9999,channel=0"}


This item would turn the Switch to **ON** every time the key was pressed

    Switch KeyPressedShortMichael "Key Michael" <present> {akm868="id=9999,channel=1"}


This item would turn the Switch to **ON** every time the key was pressed for a longer time.

    Switch KeyPressedLongeMichael "Key Michael" <present> {akm868="id=9999,channel=5"}



Example rules:

```
    rule "Turn off WIFI if not at Home"
    
    when 
Item PresenceMichael changed to OFF	  
then {
    logInfo("Wifi","Wifi OFF") 
sendCommand(Power_Upstairs_Wifi, OFF)
    }
    end
```

# Limitations
- At the moment you have to have a LANT-868 Adapter. The USB868-Adapter doesn't work.