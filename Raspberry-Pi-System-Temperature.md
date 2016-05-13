## Introduction

The [[Systeminfo Binding]] enables you to read system information through Sigar. The system information provided through this library can be extended with some additional important features.

## Solution


The temperatures for both CPU and GPU can be read through a terminal command:

```
$ cat /sys/class/thermal/thermal_zone0/temp
```

The CPU temperature is returned as millidegrees Celsius. (I'm not sure if another OS localization returns Fahrenheit)

## Requirements

* Raspberry Pi (this solution has been tested with [RPi 2B](https://www.raspberrypi.org/products/raspberry-pi-2-model-b/) running [Raspbian](https://www.raspberrypi.org/downloads/raspbian/))
* [[Exec binding]]

## Example configuration

This example gets the CPU temperature in degrees Celsius every 30 seconds and persistently stores them for presenting in a Graph. It assumes that all items for the [[Systeminfo Binding]] already are in a `systems.items` file (any other items file will do.)

add system.items file:

```
Number System_Temperature_CPU "CPU temperatuur [%.1f °C]" <temperature> (System) { exec="<[cat /sys/class/thermal/thermal_zone0/temp:30000:JS(milli.js)]" }
```

create milli.js file (in transform folder)

```
(function(i){ return i / 1000; })(input)
```

add default.sitemap file:

```
Text item=System_Temperature_CPU
```

## Sources

* [OpenHAB Community Thread](https://community.openhab.org/t/4964)