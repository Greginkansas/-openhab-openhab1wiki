**Is not included yet.. is in the pull-request phase.**

Documentation of the DD-WRT binding bundle

## Introduction

For installation of the binding, please see Wiki page [[Bindings]].

Adapt your openhab.cfg to your configuration:<BR>
*IP address of ddwrt to connect to<BR>
    ddwrt:ip=192.168.1.1<BR>
    ddwrt:port=23<BR>

*You need to configure the user and password of your ddwrt<BR>
    ddwrt:username=root<BR>
    ddwrt:password=xxxxxxx<BR>

*Interface for the 2.4 GHz wifi<BR>
    ddwrt:interface_24=ath0<BR>
*Interface for the 5 GHz wifi<BR>
    ddwrt:interface_50=ath1<BR>
*Virtuall-Interface for the guest wifi<BR>
    ddwrt:interface_guest=ath0.1<BR>


## Prepare your ddwrt device
* You have to activate the telnet connection in the ddwrt web interface.
* The changing of the telnet port in the ddwrt web interface is not always working. Test it with a telnet command shell.

## Generic Item Binding Configuration

In order to bind an item to the ddwrt device, you need to provide configuration settings. The easiest way to do so is to add some binding information in your item file (in the folder configurations/items`). 

## Switching WIFI

The following items switch WIFI, GUEST_WIFI, and the NAME of the device as string:

    String DEVICE_NAME {ddwrt="routertype"}
    Switch WIFI_24     {ddwrt="wlan24"}
    Switch WIFI_50     {ddwrt="wlan50"}
    Switch WIFI_GUEST  {ddwrt="wlanguest"}

The guest network is usually a virtuall network device. There is a bug in the ddwrt firmware. The activattion of this interface needs a workarround so it takes some seconds more as the native devices.
Tested with Archer V2 and DD-WRT v3.0-r30880 std (11/14/16)
