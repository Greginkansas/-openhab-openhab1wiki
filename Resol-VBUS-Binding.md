ResolVBUS Binding WIKI

# Introduction
This binding is for users who want to read their RESOL Controller (mostly used for solar heat) values.

For installation of the binding, please see Wiki page [[Bindings]].

# Binding configuration

First of all you need to introduce your Resol Controller in the openhab.cfg file (in the folder '${openhab_home}/configurations'). Both versions (LAN and serial/USB) are supported.

    ############################### Resol VBUS Binding ###################################
    #Either use serialport or host and port, depending on configuration
    #if the device is attached to a Resol VBUS LAN Interface provide IP and Port
    resolvbus:host=192.168.1.1
    resolvbus:port=7053
    #(optional) Password, if changed.
    #resolvbus:password=
    #if the device is attached to the serialport use this line
    #resolvbus:serialport=/dev/ttyUSB0
    # Updateinterval (optional, in seconds, defaults 1): How often should the values be updated
    resolvbus:updateinterval=10
    #(optional) the device id, if the binding doesn't detect the id itself
    #resolvbus:deviceid=

# Item Binding Configuration

In order to bind an item to the device, you need to provide configuration settings. The easiest way to do so is to add some binding information in your item file (in the folder configurations/items`). The syntax of the binding configuration strings accepted is the following:

    resolvbus="name of the value"

The **name of the value** depends on the device you're using and the values it supports. Please refer to  [this](https://docs.google.com/spreadsheets/d/1XVnwCkOXvkV6IPSh28L8li9VxwtoqopdQsMvwaSMIJk/edit?usp=sharing) page to find the right values.


A sample configuration could look like:

This item will show the value of the temperature sensor 1 of your device:

    Number TempSensor1 "Temperatur Sensor 1 [%.1f °C]" {resolvbus="Temperatur sensor 1"}


This item will show the pump speed of your pump

    Number PumpSpeed "PumpSpeed [%.0f %%]" {resolvbus="Drehzahl Relais 1"}


This item will show the system time if supported by your device.

    DateTime time "Zeit [%1$tA, %1$td.%1$tm.%1$tY]" {resolvbus="Systemzeit"}

## Tested devices

The following devices have been testes by various users and it can be confirmed that they are working

|DEVICE|DEVICE ID|
|----|---|
|Viessmann Vitosolic 200| 7321|
|Resol DeltaSol BX |7421|
|Resol DeltaSol BS/DrainBack|4278|
|Resol DeltaSol M |7311|
