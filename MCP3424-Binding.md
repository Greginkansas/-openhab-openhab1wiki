Documentation of the MCP3424 binding Bundle

## Introduction
This binding provides native access for MCP3424 18-bit delta sigma ADC
on I2C bus. Please consider datasheet for IC for future information.
Binding should work with MCP3422 and MCP3423 also, but was not tested due to lack of hardware.

**_This binding will be available from 1.9.0 onwards or is available from corresponding snapshot.
Attention: This binding is not available in download packages earlier than 1.9.0._**
Anyhow it works nicely with OpenHab 1.8.3. Simply copy jar file found on Openhab cloudbees server into
addons subdirectory and restart Openhab service.

## Generic Item Binding Configuration
Since MCP3424 is ADC converter on I2C bus, only two types of items are supported:
`Number` for raw conversion output and `Dimmer` for conversion output in percent.
Percent value is calculated dependent on set resolution. Find the example below.

# Binding Configuration in openhab.cfg
No special configuration within openhab.cfg is needed.

# Item Configuration
```
Number Test1 "Test 1" (Tests) { mcp3424="{address:6C, pin:'CH0', gain:1, resolution:12}" }
```
returns the raw conversion result on channel 0 (CH1 on datasheet) of the IC on address 0x6C
```
Dimmer Test2 "Test 2" (Tests) { mcp3424="{address:6C, pin:'CH1', gain:1, resolution:12}" }
```
returns the conversion result in percent on channel 1 (CH2 on datasheet) of the IC on address 0x6C
