Documentation of the Z-Wave binding Bundle.

## Introduction

The OpenHAB Z-Wave binding allows you to connect to your Z-Wave wireless mesh network.  A Z-Wave network typically consists of one primary controller “stick”, zero or more additional controllers and zero or more Z-Wave enabled devices, e.g. dimmers, switches, sensors etc.
Connection to the Z-Wave controller is done through the serial port of your host system. USB controllers typically create a virtual COM port to connect to the stick. Please write down the port name before configuring this binding. In case your port name changes dynamically and you want to use a symlink, see [Tricks](Samples-Tricks). A list of supported controllers is listed below.

Initialization of the binding typically takes several seconds to minutes depending on the number of devices in the network. When battery operated devices are used the binding tries to reach the device first. After one minute the node is marked sleeping. On wake up of the device initialization will continue.

[HABmin](https://github.com/cdjackson/HABmin) can be used to configure devices ([setting configuration parameters and association groups](https://github.com/cdjackson/HABmin/wiki/Z-Wave-Configuration)) directly within openHAB. Alternatively the [open-zwave control panel](https://code.google.com/p/openzwave-control-panel/) is a good choice to set up your network. Commercial software like Homeseer can be used as well.

For installation of the binding, please see Wiki page [Bindings](Bindings).

The snapshot version of the binding can be downloaded, together with the rest of openhab, from the [cloudbees](https://openhab.ci.cloudbees.com/job/openHAB1-Addons/) page.

**NOTE:** There is an issue with using the RPi with the zwave binding - see the known issues below.

**For technical reference and implementation:** Chris Jackson maintains an excellent technical resource for the ZWave binding on the [HABmin Wiki] (https://github.com/cdjackson/HABmin/wiki/Z-Wave-Configuration)
## Supported controllers

The binding supports all controllers that implement the Z-Wave Serial API. A list of confirmed supported controllers is

<table>
  <tr><td>Aeon Labs USB Z-Stick</td><td>No remarks</td></tr>
  <tr><td>Aeon Labs USB Z-Stick Gen5</td><td>Do NOT enable Soft Reset</td></tr>
  <tr><td>The Razberry-Zwave-Daughterboard</td><td>See known issues</td></tr>
  <tr><td>Vision USB stick Z-wave</td><td>Do NOT enable Soft Reset</td></tr>
  <tr><td>Z-Wave.me Z-StickC</td><td>No remarks</td></tr>
  <tr><td>Z-Wave.me ZME-UZB1</td><td>Do NOT enable Soft Reset</td></tr>
  <tr><td>Sigma UZB ZWave-Plus</td><td>No Windows drivers ?</td></tr>
</table>
**NOTE:** Sigma UZB ZWave-Plus driver can be get there https://github.com/benoit934/drivers/blob/master/ZW050x_USB_VCP_PC_Driver.zip 
This is an official driver for the RS232 chip, you just have to install it manually using the inf file, actually working on Windows 7, probably any Windows version.

## Binding Configuration

First of all you need to introduce the port settings of your Z-Wave controller in the openhab.cfg file (in the folder '${openhab_home}/configurations'). The refresh interval and the refresh delay can be specified optionally.

    ############################## Z-Wave  Binding ###################################
    
    # Z-Wave controller port
    # Valid values are e.g. COM1 for Windows and /dev/ttyS0 or /dev/ttyUSB0 for Linux
    zwave:port=COM1

<table>
<tr><td>Option</td><td>Description</td></tr>
<tr><td>zwave:port</td><td>value indicates the serial port on the host system to which the Z-Wave controller is connected, e.g. "COM1" on Windows, "/dev/ttyS0" or "/dev/ttyUSB0" on Linux or "/dev/tty.PL2303-0000103D" on Mac.<br>
Note that some controllers register themselves as a modem (/dev/ttyACM) on Linux. In this case it is necessary to add user "openhab" to the group "dialout". Else openHAB won't be able to access the controller.
</td></tr>
<tr><td>zwave:healtime</td><td>Sets the time of day when a network heal will be performed.</td></tr>
<tr><td>zwave:pollingQueue</td><td>Sets the maximum number of frames in the polling queue at once. This avoids swamping the network with poll messages.</td></tr>
<tr><td>zwave:aliveCheckPeriod</td><td>Sets the time (in milliseconds) between each node health check message. This is used to periodically check if a node is alive.</td></tr>
<tr><td>zwave:softReset</td><td>Set to true to perform a soft reset on the controller during the heal, and when the binding starts. This can help solve some problems with the stick, but it can also cause some new controllers to lock up (eg the ZWave Plus controllers)</td></tr>
<tr>
<td>zwave:masterController</td>
<td>This option tells the binding that it is the main controller in the network.
This isn't necessarily the same as a primary controller - it simply means that your openhab binding is being used
as the main network interface. If set to ```true```, the binding will configure devices automatically to send some
communications to the binding. This would include setting the wakeup class to send notifications to openhab, and set some associations so that the binding recieves notifications of configuration change or alarms.
(being introduced for 1.7)
</td>
<tr>
<td>zwave:setSUC=true</td>
<td>See: https://github.com/cdjackson/HABmin/wiki/ZWave-SUC-Controller-Mode
</td>
</table>

## Item configuration

In order to bind an item to a Z-Wave device, you need to provide configuration settings. The easiest way to do so is to add some binding information in your item file (in the folder configurations/items`). The syntax for the Z-Wave binding configuration string is explained here:
The format of the binding configuration is simple and looks like this:

    zwave="<nodeId>[:<endpointId>][:command=<command>[,parameter=<value>][,parameter=<value>]...]"

where parts in brackets indicate an optional item. Usually only one item is bound to a device, but more items can be bound to a device as well, either for reporting variables, or in case the device consists of multiple endpoints / instances.

The node ID indicates the number (in decimal notation) of the node, to which this item is bound. To find out your devices nodeIds either look at the startup log of openhab or use other Z-Wave configuration programs like openzwave control panel to detect and configure your setup.

The endpoint ID is only required/allowed when using the multi_instance command class. In case a node consists of multiple instances or endpoints, the instance number can be specified using this value. The default value is not to use the multi_instance command class - the number must be positive and must not be 0. An example of a multi-endpoint device is the Fibaro FGS 221 double relay.

**PLEASE NOTE THAT THE ENDPOINT NUMBERING CHANGED FOR 1.6 RELEASE.**
Previously, the default value was 1, but this caused problems with multi-instance devices. This meant that a binding string of ```zwave=18:1``` and ```zwave=18``` was the same - this is now NOT the case.

If you're not sure about endpoint numbering, look in the logs to see if sensor data is being correlated. You may see the following warning -:
```
2014-12-14 12:51:17.065 WARN  o.o.b.z.i.ZWaveActiveBinding[:459]- NODE 8: No item bound for event, endpoint = 0, command class = SENSOR_MULTILEVEL, value = 51, ignoring.
```

This indicates that the binding can’t find an item linked to this sensor - often this is because the endpoint numbering is incorrect. If the warning above says endpoint = 0, then the binding string shouldn't include an endpoint number.


The command is optional, but recommended if you have multiple items bound to the same device, or the device reports multiple bits of information. Without the command class, the binding can not unambiguously differentiate different data, so it is recommended to provide a command class. Z-Wave nodes support functionality through command classes. A specific command class can be specified to use that specific functionality of the node. A node can contain multiple supported command classes. If the command is omitted, the best suitable command class for the item / node combination is automatically chosen.

Command classes may support parameters. A parameter is a name=value pair that configures some aspect of the command class on the node or in the binding.

A list with command classes and parameter values is provided below:

## Supported Command Classes

Each node in the network provides functionality in the form of Command Classes. The OpenHAB Z-Wave binding implements the same Command Classes to be able to use the nodes in the network. Not all Z-Wave Command classes are currently supported by the binding. The supported command classes are listed in the table below.

<table>
  <tr><td><b>Command Class</b></td><td><b>Remarks</b></td><td><b>Supported parameters</b></td></tr>
  <tr><td>NO_OPERATION</td><td>Used by the binding during initialization</td><td></td></tr>
  <tr><td>BASIC</td><td>Provides basic SET and GET of the default node value</td><td></td></tr>
  <tr><td>HAIL</td><td>Used by nodes to indicate that they want to be polled. The binding handles this automatically</td><td></td></tr>
  <tr><td>METER</td><td>Used to get measurements from a node</td><td>**meter_scale=value** :  optional parameter to select the meter scale in case the meter supports multiple scales (and types). Value is one of the following **textual** values:<br/>E_KWh (0, MeterType.ELECTRIC, "kWh", "Energy") <br/>E_KVAh (1, MeterType.ELECTRIC, "kVAh", "Energy")<br/>E_W(2, MeterType.ELECTRIC, "W", "Power")<br/>E_Pulses (3, MeterType.ELECTRIC, "Pulses", "Count")<br/>E_V (4, MeterType.ELECTRIC, "V", "Voltage")<br/>E_A (5, MeterType.ELECTRIC, "A", "Current")<br/>E_Power_Factor (6, MeterType.ELECTRIC, "Power Factor", "Power Factor")<br/>G_Cubic_Meters (0, MeterType.GAS, "Cubic Meters", "Volume")<br/>G_Cubic_Feet (1, MeterType.GAS, "Cubic Feet", "Volume")<br/> G_Pulses(3, MeterType.GAS, "Pulses", "Count")<br/>W_Cubic_Meters (0, MeterType.WATER, "Cubic Meters", "Volume")<br/>W_Cubic_Feet (1, MeterType.WATER, "Cubic Feet", "Volume")<br/>W_Gallons (2, MeterType.WATER, "US gallons", "Volume")<br/>W_Pulses (3, MeterType.WATER, "Pulses", "Count")</td></tr>
   <tr><td>METER_RESET</td><td>Used to reset a meter back to 0</td><td>To reset a meter use the "meter_reset=true" attribute on the meter. 
   When moving the switch from off to on it will reset the meter.<br/>e.g.:<br/>
Switch sReset { zwave="8:command=meter, meter_reset=true"}
   </td>
</tr>
  <tr><td>SWITCH_BINARY</td><td>Used to bind directly to a SWITCH</td><td></td></tr>
  <tr><td>SWITCH_MULTILEVEL</td><td>Used to bind directly to a DIMMER</td><td>restore_last_value=true : restores the dimmer to it's last value if an ON command is sent to the dimmer (as opposed to setting it's value to 100%)</td></tr>
  <tr><td>SENSOR_BINARY</td><td>Used to bind to a sensor.</td><td>**sensor_type=value** : optional parameter to select a sensor in case the node supports multiple sensors. Value is one of the following **numerical** values:<br/>
	1 = General Purpose<br/>
	2 = Smoke<br/>
	3 = Carbon Monoxide<br/>
	4 = Carbon Dioxide<br/>
	5 = Heat<br/>
	6 = Water<br/>
	7 = Freeze<br/>
	8 = Tamper<br/>
	9 = Aux<br/>
	10 = Door/Window<br/>
	11 = Tilt<br/>
	12 = Motion<br/>
	13 = Glass Break<br/>
</td></tr>
  <tr><td>SENSOR_MULTILEVEL</td><td>Used to bind to e.g. a temperature sensor. Currently only single sensors are supported.</td><td>**sensor_type=value** : optional parameter to select a sensor in case the node supports multiple sensors. Value is one of the following **numerical** values:<br/>
1 = Temperature<br/>
2 = General<br/>
3 = Luminance<br/>
4 = Power<br/>
5 = RelativeHumidity<br/>
6 = Velocity<br/>
7 = Direction<br/>
8 = AtmosphericPressure<br/>
9 = BarometricPressure<br/>
10 = SolarRadiation<br/>
11 = DewPoint<br/>
12 = RainRate<br/>
13 = TideLevel<br/>
14 = Weight<br/>
15 = Voltage<br/>
16 = Current<br/>
17 = CO2<br/>
18 = AirFlow<br/>
19 = TankCapacity<br/>
20 = Distance<br/>
21 = AnglePosition<br/>
22 = Rotation<br/>
23 = WaterTemperature<br/>
24 = SoilTemperature<br/>
25 = SeismicIntensity<br/>
26 = SeismicMagnitude<br/>
27 = Ultraviolet<br/>
28 = ElectricalResistivity<br/>
29 = ElectricalConductivity<br/>
30 = Loudness<br/>
31 = Moisture<br/>
32 = MaxType
</td></tr>
  <tr><td>MULTI_INSTANCE</td><td>Used to channel commands to the right endpoint on multi-channel devices. See item configuration.</td><td></td></tr>
  <tr><td>MANUFACTURER_SPECIFIC</td><td>Used to get manufacturer info from the device</td><td></td></tr>
  <tr><td>BATTERY</td><td>Used to get the battery level from battery operated devices. See item configuration.</td><td></td></tr>
  <tr><td>WAKE_UP</td><td>Used to respond to wake-up signals of battery operated devices.</td><td></td></tr>
  <tr><td>VERSION</td><td>Used to get version info from a node.</td><td></td></tr>
  <tr><td>SENSOR_ALARM</td><td>Used to get alarm info from sensors.</td><td>*alarm_type=value* : optional parameter to select an alarm type in case the node supports multiple alarms. Value is one of the following **numerical** values: <br/>GENERAL(0, "General")<br/>SMOKE(1, "Smoke")<br/>CARBON_MONOXIDE(2, "Carbon Monoxide")<br/>CARBON_DIOXIDE(3, "Carbon Dioxide")<br/>HEAT(4, "Heat")<br/>FLOOD(5, "Flood")</td></tr>
  <tr><td>SCENE_ACTIVATION</td><td>Used to respond to Scene events.</td><td>scene=xx to identify the scene to trigger on <br/> state=xx to set the item to the specified state (xx is an integer)</td></tr>
  <tr><td>ALARM</td><td></td></tr>
  <tr><td>MULTI_CMD</td><td>Used to send multiple command classes in a single packet.</td><td></td></tr>
  <tr><td>THERMOSTAT_MODE</td><td>(since 1.6.0) Used to get and set the mode (of Number type) of the thermostat</td><td> numeric values translate to the following types <br/>
                0 = "Off"<br/>
		1 = "Heat"<br/>
		2 = "Cool"<br/>
		3 = "Auto"<br/>
		4 = "Aux Heat"<br/>
		5 = "Resume"<br/>
		6 = "Fan Only"<br/>
		7 = "Furnace"<br/>
		8 = "Dry Air"<br/>
		9 = "Moist Air"<br/>
		10 = "Auto Changeover"<br/>
		11 = "Heat Econ"<br/>
		12 = "Cool Econ"<br/>
		13 = "Away"<br/>
  </td></tr>
 <tr><td>THERMOSTAT_OPERATING_STATE</td><td>(since 1.6.0) Used to get the operating state (of Number type) of the thermostat</td><td> numeric values translate to the following types <br/>
                0 = "Idle"<br/>
		1 = "Heating"<br/>
		2 = "Cooling"<br/>
		3 = "Fan Only"<br/>
		4 = "Pending Heat"<br/>
		5 = "Pending Cool"<br/>
		6 = "Vent / Economizer"<br/>
  </td></tr>
<tr><td>THERMOSTAT_SETPOINT</td><td>(since 1.6.0) Used to get and set the setpoint of the thermostat</td><td>**setpoint_type=value** : parameter to select setpoint type, value is one of the following numerical values:<br/>
               1 = Heat <br/>
               2 = Cool <br/>
 *setpoint_scale=value** : parameter to select setpoint scale, value is one of the following numerical values:<br/>
               0 = Celsius <br/>
               1 = Fahrenheit <br/>
</td></tr>
<tr><td>THERMOSTAT_FAN_MODE</td><td>(since 1.6.0) Used to get the fan mode (of Number type) of the thermostat</td><td> numeric values translate to the following types <br/>
                0 = "Auto Low"<br/>
		1 = "On Low"<br/>
		2 = "Auto High"<br/>
		3 = "On High"<br/>
		4 = "Unknown"<br/>
		5 = "Unknown"<br/>
		6 = "Circulate"<br/>
  </td></tr>
<tr><td>THERMOSTAT_FAN_STATE</td><td>(since 1.6.0) Used to get the fan state (of Number type) of the thermostat</td><td> numeric values translate to the following types <br/>
                0 = "Idle"<br/>
		1 = "Running"<br/>
		2 = "Running High"<br/>
  </td></tr>
<tr><td>CONFIGURATION</td>
<td>Used to set configuration parameters. Normally, this is done through HABmin as most configuration is static, but some devices have parameters that need to be changed via a rule or sitemap. </td>
<td>Use the "parameter=" option in the binding string to set the parameter number linked to this item.</td>
</tr>
<tr><td>INFO</td><td>This is not a 'real' zwave command class, but can be used to get information from the binding about a node and its state.</td>
<td>
Controller only:<br/>
HOME_ID<br/>
SOF<br/>
CAN<br/>
NAK<br/>
OOF<br/>
ACK<br/>
TIME_OUT<br/>
TX_QUEUE<br/>
<br/>
All Nodes:<br/>
NODE_ID<br/>
LISTENING<br/>
DEAD<br/>
ROUTING<br/>
VERSION<br/>
BASIC<br/>
BASIC_LABEL<br/>
GENERIC<br/>
GENERIC_LABEL<br/>
SPECIFIC<br/>
SPECIFIC_LABEL<br/>
MANUFACTURER<br/>
DEVICE_ID<br/>
DEVICE_TYPE<br/>
LAST_UPDATE<br/>
</td></tr>
<tr><td>INDICATOR</td>
<td>Show the state, or level of a device, usually through a button LED, or display on the actual device</td>
<td>If you use the bit=n (n between 0 and 8) parameter then you can bind Switch Items to individual bits in the indicator value which can be used to turn on and off status LEDs on device buttons for instance.</td>
</table>

## Parameters that can be added to any item

There are some general parameters that can be added to any command class
in an item string. These are `refresh_interval=value` and `respond_to_basic=true`

`refresh_interval=value` sets the refresh interval to *value* seconds. 0 indicates that no polling is performed and the node should inform the binding itself on value changes. This is the default value.

`respond_to_basic=true` indicates that the item will respond to basic reports. Some Fibaro contacts and universal sensors report their values as BASIC reports instead of a specific command class. You can add this parameter to an item to indicate that this item should respond to those reports.

`meter_zero=xx.x` can be set when using the Meter command class. If set, anything below the value specified will be considered as 0. This allows the user to account for small power consumption readings even when a device is off.

`sensor_scale=X` can be set for the multilevel sensor class to force the sensor to be converted to a specific scale. This uses the scale information provided by the device to decide how, or if, the conversion needs to be applied. Currently this is only available for temperature sensors. eg. `sensor_scale=0` will ensure that a temperature sensor is always shown in celsius, while `sensor_scale=1` will ensure fahrenheit.

`invert_state=true` can be used to invert the state of a multilevel switch command class. eg this can be used to reverse the direction of a rollershutter.

### Basic command class

The basic Command Class is a special command class that almost every node implements. It provides functionality to set a value on the device and/or to read back values. It can be used to address devices that are currently not supported by their native command class like thermostats.

When the basic command class is used, devices support setting values and reporting values when polling. Direct updates from the device on changes will fail however.

You can force a device to work with the basic command set (or any specific command set for that matter using a syntax like:

    Switch    ZwaveDevice        { zwave="3:1:command=BASIC" }


To find out which command classes are supported by your Z-Wave device, you can look in the manual or use the list at http://www.pepper1.net/zwavedb/ or http://products.z-wavealliance.org/. In case your command class is supported by the device and binding, but you have a problem, you can create an issue at: https://github.com/openhab/openhab/issues. In case you want a command class implemented by the binding, please create an issue.

## Known Issues
ARM Compatibility issue (fixed - updated Dec. 11, 2016 see below):
There seems to be an issue with the binding running on the latest oracle VM Beta, on ARM based architectures (e.g. raspberry PI). It manifests itself as messages being received multiple times and causes considerable problems with the operation of the binding. In large networks, the queue can get extremely long, which can delay initialisation considerably and cause potentially long delays in sending messages. Some time has been spent investigating this issue and a solution has not been found - the issue doesn't appear to be with the binding itself as the problem doesn't manifest itself on an other platform. If anyone with the hardware and programming experience can help with this it would be useful (add information to https://github.com/openhab/openhab/issues/1564).

(Update Dec. 11, 2016): This issue is no longer affecting "all" ARM processors. The above issue report has been closed as the problem has vanished in newer Java JRE versions.  
I recently built a successful setup with no duplication problems with these versions of software and hardware:  
Raspberry Pi 3 (ARM v7l)
Aeon Zstick Gen5
Raspbian - November 2016 - Kernel 4.4 
Openhab 1.8.3 
Z-wave binding version 1.8.3
Java Runtime SE: build 1.8.0_65-b17
 
The original issue (1564) has been closed and the symptoms have vanished since an update to Java JRE's serial library that is now standard in the new versions of Raspbian. I have not tested with original Raspberry pi (ARM v5) nor a Raspberry pi 2 (ARM v6). Assuming it was fixed in Java those should work now too. But maybe it is only fixed based on Raspberry pi 3 being ARM v7l.

You should no longer fear using OpenHAB with raspberry pi 3 and Z-wave! For my install it is very successful and zero errors.

## Examples

Numerous examples of item definitions available at the [Binding examples page](Z-Wave-Binding-Examples)

## Healing
The binding can perform a nightly heal. This will try to update the neighbor node list, associations and routes. The actual routing is performed by the controller.
Two options are possible in the config file to set the healing. ```healtime``` sets the time (in hours) that the automatic heal will occur. ```softReset``` can also be set to true to perform a soft reset on the controller before the heal. Note that it has been observed that this can cause some zwave-plus sticks to lock up, so you should test this first.

## Battery Devices and Wakeup
One of the questions that gets asked a lot is _"my battery device is sending information, so it's clearly awake, but I can't change parameters"_. ZWave battery devices only 'wake up' periodically however they will send their data whenever THEY want. So if the door opens, then a sensor will send its data immediately (otherwise it would be of no use!) and for temperature sensors, or multi sensors, there's normally a configuration that allows you to configure how often the sensor will send the data, or how much change there needs to be before it will send an update (or a combination of the two).

However, wakeup is different. In the above scenarios, the device is not 'awake' - it's just sending notifications. When a device wakes up, it sends a special message to the binding to say "I'm awake, do you have anything to send me". We can then send data to the device (configuration setting, parameter requests etc), and the device can respond. When we have no more messages to send to the device, we tell it to go back to sleep (this currently happens 1 second after we send the last message - just to give the system time to respond to any rules etc before the device goes to sleep again).

A device will wake up based on the information in the wakeup command class, which is available in HABmin. If wakeup isn't set correctly, then we can't command the device, even though we may receive sensor data etc. The binding will attempt to configure this automatically when the device/binding initialises - it will make sure that the node is set to reference the binding, but in general it won't touch the time. The exception to this is where the time is set to 0, it will set it to 1 hour.

If wakeup hasn't been set, then we will never be able to send commands to the device to initialise it, or change parameters. In this case, you need to wake the device up manually. Normally, this is achieved by pressing a button on the device (maybe 3 times). This causes the device to send what's called a NIF (Node Information Frame) - this is sent as a broadcast though so it's not routed. This means that the device MUST be able to communicate directly to the controller, so it needs to be close by.

The last wakeup time is shown in HABmin in the wakeup area for a battery device.

One last point on the wakeup configuration node. There is (currently) no way to set the target node - the binding will automatically set it to its own ID when the interval is changed. Some people have used a value of 255 for the target node, or it may come as the default value - this means that the wakeup is broadcast to anyone who wants to listen. This might seem a good idea as multiple controllers can receive the message however, broadcast messages do not get routed, so this only works if the controller is in direct contact with the device which is often not the case.


## Database

**Note**: A new online database editor is being produced. This should make it easier for most people to add and update devices within the database, and will improve support for both the OH1 and OH2 bindings. Please support this [here](http://www.cd-jackson.com/index.php/zwave/zwave-device-database/zwave-device-database-guide).

The binding uses a database of devices so that it can work around any quirks, or present information about association groups and configuration data. The format for the database is [here](https://github.com/cdjackson/HABmin2/wiki/Z-Wave-Product-Database).

If you are not able to produce the XML file yourself, then please open an issue. The following information is required -:
* Type and ID for the device - HABmin will print this information when a device is not in the database
* Manufacturer
* Reference to the device manual (ie link to PDF)
* Link to device in pepper1 database (if it exists). [http://www.pepper1.net/zwavedb/](http://www.pepper1.net/zwavedb/)

## Logging

The ZWave binding can generate a lot of debug/trace logging when enabled so it is advisable to generate a separate log file for this binding. This also makes it easier for analysis since there is nothing from other bindings/openHAB polluting the logs.

In order to configure logging for this binding to be generated in a separate file add the following to your /configuration/logback.xml file;
```xml
<appender name="ZWAVEFILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
   <file>logs/zwave.log</file>
   <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- weekly rollover and archiving -->
      <fileNamePattern>logs/zwave-%d{yyyy-ww}.log.zip</fileNamePattern>
      <!-- keep 30 days' worth of history -->
      <maxHistory>30</maxHistory>
   </rollingPolicy>
   <encoder>
     <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level] [%-30.30logger{36}:%-4line]- %msg%n</pattern>
   </encoder>
</appender>
    
<!-- Change DEBUG->TRACE for even more detailed logging -->
<logger name="org.openhab.binding.zwave" level="DEBUG" additivity="false">
   <appender-ref ref="ZWAVEFILE" />
</logger>
```
It is highly recommended to turn on at least DEBUG logging whilst setting up and configuring your ZWave network for the first time. 


## Refreshing a value manually
In seldom cases it is useful to refresh values from the network manually in rules.

To achieve this, use the refresh.REFRESH command on the corresponding item.
e.g.

````sendCommand("BEDROOM_TV", refresh.REFRESH)````

Attention:
This works only with OH2
