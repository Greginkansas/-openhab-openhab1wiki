Documentation of the PLCLogo binding Bundle

## Introduction

This binding provides native support of Siemens Logo! PLC devices. Communication with Logo is done via Moka7 library.
Binding supports currently three types of items: `Contact` for digital inputs, `Switch` for digital outputs
and `Number` for analog inputs/outputs. Currently only two devices are supported: 0BA7 (Logo! 7) and 0BA8 (Logo! 8).
Reading is supported from follow blocks on both devices: `I`, `Q`, `M`, `AI`, `AQ` and `AM`. On Logo! 8 `NI`, `NQ`, `NAI` and `NAQ` are supported additionally. Writing to the PLC can be done via `VB` and `VW` for both devices.
Use `Switch` items for writing of digital and `Number` of analog values. Binding works nicely at least 100ms polling rate. Additionally multiple devices are supported. Different families of Logo! devices should work also, but was not testen now due to lack of hardware. Find examples below.

**_This binding will be available from 1.9.0 onwards or is available from corresponding snapshot_**

## Generic Item Binding Configuration

# Binding Configuration in openhab.cfg

# Item Configuration
```
Switch ReadOutputQ13 {plclogo="bcu:Q13"}
```
read set value on Q13 from Logo! device named 'bcu'

```
Switch ReadWriteBinaryValue {plclogo="bcu:VB0.0"} }
```
read/set binary input block mapped to VB address 0, bit 0

```
Number ReadWriteAnalogValue {plclogo="bcu:VW5"}
```
read/set analog input block mapped to VW address 5
