## Introduction

The [[Systeminfo Binding]] enables you to read system information through Sigar. The system information provided through this library can be extended with some additional important features.

## Solution

This solution was developed for a [Raspberry Pi 2 model B](https://www.raspberrypi.org/products/raspberry-pi-2-model-b/) running [Raspbian](https://www.raspberrypi.org/downloads/raspbian/)) and an [[apt-get installation|Linux-and-OS-X#apt-get]].

The temperatures for both CPU and GPU can be read through terminal commands:

```
$ cat /sys/class/thermal/thermal_zone0/temp
46540
$ /opt/vc/bin/vcgencmd measure_temp
temp=47.1'C
```

This means the output can be captured using the [[Exec binding]]. 

However. The CPU temperature is returned as millidegrees Celsius. (I'm not sure if another OS localization returns Fahrenheit). The GPU temperature has some surrounding text for readability. Both need some [[transformations]] to get the values into OpenHAB properly.

The CPU temperature is computed through a Javascript transformation.
The GPU temperature is captured through a Regex transformation.

## Requirements and prerequisites

* Raspberry Pi running OpenHAB 
* The [[Exec binding]] has already been installed.
* User `openhab` is member of the `video` group

### Add openhab user to video group:

The user `openhab` needs to be member of the `video` group to be able to run the `vcgencmd` command. Otherwise you will see a [VCHI initialization failed](http://raspberrypi.stackexchange.com/questions/7546/munin-node-plugins-vchi-initialization-failed) error message.

Group memberships are only updated after a reboot.

```
$ sudo usermod -a -G video openhab
$ sudo reboot
```

## Example configuration

This example gets the CPU temperature in degrees Celsius every 30 seconds and persistently stores them for presenting in a graph. It assumes that all items containing system information (potentially including those for the [[Systeminfo Binding]]) are in a `systems.items` file (any other items file will do).

Create a `milli.js` file (in transform folder) with this content:

```
(function(i){ return i / 1000; })(input)
```

Add to `system.items` file:

```
// System temperatures
Group  System_Temperature_Chart (System, Charts)
Number System_Temperature_Chart_Period "Periode" (System)
Number System_Temperature_CPU "Temperature CPU [%.1f °C]" <temperature> (System_Temperature_Chart) { exec="<[cat /sys/class/thermal/thermal_zone0/temp:60000:JS(milli.js)]" }
Number System_Temperature_GPU "Temperature GPU [%.1f °C]" <temperature> (System_Temperature_Chart) { exec="<[/opt/vc/bin/vcgencmd measure_temp:60000:REGEX(temp=(.*?)'C)]" }
```

Add to `rrd4j.persist` file:

```
System_Temperature_Chart* : strategy = everyChange, everyMinute, restoreOnStartup
```

Add to `default.sitemap` file:

```
Text item=System_Temperature_CPU label="Temperature [%.1f °C]" {
	Frame {
		Text item=System_Temperature_CPU					
		Text item=System_Temperature_GPU
	}
	Frame {
		Switch item=System_Temperature_Chart_Period mappings=[0="1h", 1="4h", 2="8h", 3="12h", 4="24h"]
		Chart  item=System_Temperature_Chart period=h   refresh=60000 visibility=[System_Temperature_Chart_Period==0, System_Temperature_Chart_Period=="Uninitialized"]
		Chart  item=System_Temperature_Chart period=4h  refresh=60000 visibility=[System_Temperature_Chart_Period==1]
		Chart  item=System_Temperature_Chart period=8h  refresh=60000 visibility=[System_Temperature_Chart_Period==2]
		Chart  item=System_Temperature_Chart period=12h refresh=60000 visibility=[System_Temperature_Chart_Period==3]
		Chart  item=System_Temperature_Chart period=D   refresh=60000 visibility=[System_Temperature_Chart_Period==4]
	}
}
```

## Sources

* [OpenHAB Community Thread](https://community.openhab.org/t/4964)