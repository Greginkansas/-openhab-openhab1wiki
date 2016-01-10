# Introduction
This binding integrates [PiXtend](http://www.pixtend.de) Raspberry- / Banana-Pi interface board into openHAB.

It is based on [PiXtend4J](https://github.com/kwave/pxdev) and is intended to be used with the original (or a compatible) Firmware. To be able to use this Binding, you need to run openHAB on the Pi physically connected to your PiXtend board. If you prefer running your main openHAB Instance on another host, you can connect the PiXtend-OpenHAB as slave instance (e.g. via MQTT-Eventbus Binding).

# Configuration
Currently there are no global configuration options (openhab.cfg) available.

You can bind any Input or Output of the PiXtend to an openHAB Item. As Inputs and / or Outputs may also be "virtual", they will be called "Data Port" within this document.

To bind an Item to a data-port, use the port Identifier. Identifiers that can be read are always published as openHAB-states, whereas identifiers that support write only listen to openHAB-commands.

The following Identifiers are currently available:

    Identifiers:   AI0 AI1 AI2 AI3
    Description:   Analog Inputs 0-3
    Supports:      Read
    Data Type:     Decimal

    Identifiers:   DI0 DI1 DI2 DI3 DI4 DI5 DI6 DI7
    Description:   Digital Inputs 0-7
    Supports:      Read
    Data Type:     Open / Closed (=High/Low)

    Identifiers:   GPIO0 GPIO1 GPIO2 GPIO3
    Description:   Analog Inputs 0-3
    Supports:      Read & Write
    Data Type:     Read: Open / Closed (=High/Low), Write: ON / OFF

    Identifiers:   REG_STATUS
    Description:   Uc Status Register
    Supports:      Read
    Data Type:     Decimal

    Identifiers:   FIRMWARE_VERSION
    Description:   Uc Firmware version
    Supports:      Read
    Data Type:     String

    Identifiers:   RESET_UC
    Description:   Performs a Uc Reset (only if "ON" is received)
    Supports:      Write
    Data Type:     ON/OFF

    Identifiers:   AO0 AO1
    Description:   Analog Outputs 0-1
    Supports:      Write
    Data Type:     Decimal

    Identifiers:   DO0 DO1 DO2 DO3 DO4 DO5
    Description:   Digital Outputs 0-5
    Supports:      Write
    Data Type:     ON / OFF

    Identifiers:   REL0 REL1 REL2 REL3
    Description:   Digital Outputs 0-3
    Supports:      Write
    Data Type:     ON / OFF


To bind your Item to a DataPort simply use the Identifier as binding String:

    Switch "PiXtend Relais 1" {pixtend=REL1}
    String "Pixtend Firmware Version" {pixtend= FIRMWARE_VERSION}

