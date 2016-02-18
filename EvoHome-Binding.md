## Introduction

This binding supports the EvoHome products from Honeywell. It isn't currently in the main distribution (and as it is only OH1 it is unlikely to get merged) but you can found details on where to get the JAR in the forums: 

https://community.openhab.org/t/new-evohome-binding/7696

## Binding Configuration

First you need to configure the following values in the openhab.cfg file (in the folder '${openhab_home}/configurations'). The defaults should suffice unless you know what you are doing.

    ################################### EvoHome Binding #####################################
    #
    evohome:username=<username>
    evohome:password=<password>
    evohome:applicationId=91db1612-73fd-4500-91b2-e63b069b185c
    evohome:refresh=600000

## Item Binding Configuration

To configure the items you need the name you have configured either in the Andriod/iPhone app or on the controller. 

The EvoHome binding works on the concept of giving each item a type. This will determine the value that item is loaded with when an update is received.

Valid types are

<table>
<tr><td><b>EvoHome Type</b></td><td><b>OpenhabType</b></td><td><b>Read/Write</b></td><td><b>From Version</b></td></tr>
<tr><td>LOCATION_NAME</td><td>String</td><td>Read</td><td>1.9.0</td>
<tr><td>LOCATION_ID</td><td>Number</td><td>Read</td><td>1.9.0</td>
<tr><td>WEATHER_CONDITION</td><td>String</td><td>Read</td><td>1.9.0</td>
<tr><td>WEATHER_TEMPERATURE</td><td>Number</td><td>Read</td><td>1.9.0</td>
<tr><td>WEATHER_HUMIDITY</td><td>Number</td><td>Read</td><td>1.9.0</td>
<tr><td>WEATHER_UNIT</td><td>String</td><td>Read</td><td>1.9.0</td>
<tr><td>WEATHER_PHRASE</td><td>String</td><td>Read</td><td>1.9.0</td>
<tr><td>THERMOSTAT_TEMPERATURE</td><td>Number</td><td>Read</td><td>1.9.0</td>
<tr><td>THERMOSTAT_SETPOINT_VALUE</td><td>Number</td><td>Read</td><td>1.9.0</td>
<tr><td>DEVICE_NAME</td><td>String</td><td>Read</td><td>1.9.0</td>
<tr><td>DEVICE_ID</td><td>Number</td><td>Read</td><td>1.9.0</td>
<tr><td>THERMOSTAT_UNIT</td><td>String</td><td>Read</td><td>1.9.0</td>
<tr><td>THERMOSTAT_MODE</td><td>String</td><td>Read</td><td>1.9.0</td>
<tr><td>THERMOSTAT_SETPOINT_STATUS</td><td>String</td><td>Read</td><td>1.9.0</td>
<tr><td>THERMOSTAT_SETPOINT_NEXTTIME</td><td>DateTime</td><td>Read</td><td>1.9.0</td>
</table>

Examples, configure for your items:
================

    String EvoHome_Name "EvoHome Location Name [%s]" { evohome="locationName=LOCATION_NAME,type=LOCATION_NAME" }
    Number EvoHome_Id "EvoHome Location ID [%s]" { evohome="locationName=LOCATION_NAME,type=LOCATION_ID" }
    String EvoHome_Weather_Condition "Weather Condition [%s]"  { evohome="locationName=LOCATION_NAME,type=WEATHER_CONDITION" }
    Number EvoHome_Weather_Temp "Weather Temp [%.1f °C]"  { evohome="locationName=LOCATION_NAME,type=WEATHER_TEMPERATURE" }
    Number EvoHome_Weather_Humidity "Weather Humidity [%.1f %%]"  { evohome="locationName=LOCATION_NAME,type=WEATHER_HUMIDITY" }
    String EvoHome_Weather_Unit "Weather Unit [%s]"  { evohome="locationName=LOCATION_NAME,type=WEATHER_UNIT" }
    String EvoHome_Weather_Phrase "Weather Phrase [%s]" { evohome="locationName=LOCATION_NAME,type=WEATHER_PHRASE" }
    Number Bedroom_Radiator_Current_Temp    "Bedroom Radiator Temp [%.1f °C]" { evohome="locationName=LOCATION_NAME,deviceName=DEVICE_NAME,type=THERMOSTAT_TEMPERATURE" }
    Number Bedroom_Radiator_Target_Temp     "Bedroom Radiator Target Temp [%.1f °C]" { evohome="locationName=LOCATION_NAME,deviceName=DEVICE_NAME,type=THERMOSTAT_SETPOINT_VALUE" }
    String Bedroom_Radiator_Device_Name "Bedroom Radiator Name [%s]" { evohome="locationName=LOCATION_NAME,deviceName=DEVICE_NAME,type=DEVICE_NAME" }
    Number Bedroom_Radiator_Device_Id "Bedroom Radiator Id [%s]" { evohome="locationName=LOCATION_NAME,deviceName=DEVICE_NAME,type=DEVICE_ID" }
    String Bedroom_Radiator_Unit "Bedroom Radiator Unit [%s]" { evohome="locationName=LOCATION_NAME,deviceName=DEVICE_NAME,type=THERMOSTAT_UNIT" }
    String Bedroom_Radiator_Mode "Bedroom Radiator Mode [%s]"  { evohome="locationName=LOCATION_NAME,deviceName=DEVICE_NAME,type=THERMOSTAT_MODE" }
    String Bedroom_Radiator_Set_Status "Bedroom Radiator Set Status [%s]"  { evohome="locationName=LOCATION_NAME,deviceName=DEVICE_NAME,type=THERMOSTAT_SETPOINT_STATUS" }
    DateTime Bedroom_Radiator_Set_NextTime "Bedroom Radiator Set Time [%1$tT, %1$tF]"  { evohome="locationName=LOCATION_NAME,deviceName=DEVICE_NAME,type=THERMOSTAT_SETPOINT_NEXTTIME" }

================