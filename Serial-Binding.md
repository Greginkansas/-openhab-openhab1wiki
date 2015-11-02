## Introduction

For installation of the binding, please see Wiki page [[Bindings]].

## Generic Item Binding Configuration

In order to bind an item to a Serial device, you need to provide configuration settings. The easiest way to do so is to add some binding information in your item file (in the folder configurations/items`). 

The format of the binding configuration is simple and looks like this:

    serial="<port>@<baudrate>" 
    serial="<port>@<baudrate>,REGEX(<regular expression>)" 

* where `<port>` is the identification of the serial port on the host system, e.g. "COM1" on Windows, "/dev/ttyS0" on Linux or "/dev/tty.PL2303-0000103D" on Mac.
* where `<baudrate>` is the baud rate of the port. Backward compability is given, as if no baud rate is specified  the serial binding defaults to 9600 bauds.
* where `REGEX(<regular expression>)` allows to parses for special strings or numbers in the serial stream. This is based on the [RegEx Service](https://github.com/openhab/openhab/wiki/Transformations#regex-transformation-service). This is optional and new in version 1.8. 

Switch items with this binding will receive an ON-OFF update on the bus, when ever data becomes available on the serial interface (or simply by short-cutting pins 2 and 7 on the RS-232 interface)

String items will receive the submitted data in form of a string value as a status update, while openHAB commands to a String item is sent out as data through the serial interface.

Number items will receive the RegEx result and tries to convert the string to a number.

As a result, your lines in the items file might look like the following:

    Switch HardwareButton     "Bell"	           (Entrance)      { serial="/dev/ttyS0" }
    String AVR                "Surround System"    (Multimedia)    { serial="/dev/ttyS1@115200" } 
    Number Temperature        "My Temp. Sensor"    (Weather)       { serial="/dev/ttyS1@115200,REGEX(ID:2.*,T:([0-9.]*))" } 

Note: If you are working with a Mac, you might need to install a driver for your USB-RS232 converter (e.g. [osx-pl2303](http://osx-pl2303.sourceforge.net/) or [pl2303](http://mac.softpedia.com/get/Drivers/PL2303-OS-X-driver.shtml)) and create the /var/lock folder, see the [rxtx troubleshooting guide](http://rxtx.qbang.org/wiki/index.php/Trouble_shooting#Mac_OS_X_users).

Note2: If you are using **non standard serial ports** you have to adopt start.sh to have the serial port included. the java command line should then include the following parameters:

```
-Dgnu.io.rxtx.SerialPorts=/dev/ttyAMA0
```

whereas `ttyAMA0` is the path to your serial port. Please be aware to change all scripts you might use for startup (debug, automatic start in linus,...)

Note3: With version 1.8 it is allowed to use a serial connection for multiple items. This has changed for the RegEx extension.