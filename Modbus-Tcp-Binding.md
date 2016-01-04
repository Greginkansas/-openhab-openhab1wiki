Documentation of the Modbus TCP binding Bundle

## Introduction

The modbus TCP binding polls the bus in an configurable interval for a configurable length.  

For installation of the binding, please see Wiki page [[Bindings]].
 
## Details

## Binding Configuration

add to `${openhab_home}/configuration/`

Modbus binding allows to connect to multiple Modbus slaves as TCP master. This implementation works with coils (boolean values) only.
Entries in openhab config file should look like below.
Most of config parameters are related to specific slaves, the only exception is

     modbus:poll=<value>

which sets refresh interval to Modbus polling service. Value is in milliseconds - optional, default is 200

     modbus:<slave-type>.<slave-name>.<slave-parameter>

     <slave-type> can be either "tcp" or "serial"
     <slave-name> is unique name per slave you are connecting to.
     <slave-parameter> are pairs key=value


Valid keys are

<table>
  <tr><td>connection</td><td>mandatory</td><td>for tcp connection use form host_ip[:port] e.g. 192.168.1.55 or 192.168.1.55:511. If you omit port, default 502 will be used. For serial connections use just COM port name optional [:baud:dataBits:parity:stopBits:encoding] <br/>opitons are: parity={even,odd}; encoding={ascii,rtu,bin} (default is ascii, supported since v1.7)</td></tr>
  <tr><td>id</td><td>optional</td><td>slave id, default 1</td></tr>
  <tr><td>start</td><td>optional</td><td>slave start address, default 0</td></tr>
  <tr><td>length</td><td>mandatory</td><td>number of data item to read, default 0 (but set it to something meaningful :)</td></tr>
  <tr><td>type</td><td>mandatory</td><td>data type, can be either "coil", "discrete", "holding", "input" or "register", now only "coil", "discrete", "holding" and "input" are supported</td></tr>
</table>

Remark : in "`openhab_default.cfg`", the modbus binding section has a wrong key "`host`", this doesn't work if you put your slave ip address here. So you have to replace "`host`" by "`connection`" witch is the right key as mentioned above.

Modbus read functions 
- `type=coil` uses function 1,
- `type=discrete` uses function is 2,
- `type=holding` uses function is 3,
- `type=input` uses function ist 4

Modbus write functions 
- `type=coil` uses function 5,
- `type=holding` uses function is 6,
 see also http://www.simplymodbus.ca

with `type=holding` and `type=input` you can now only operate with datatype byte!!!
see point 4 below

Minimal construction in openhab.config for TCP connections will look like:

    modbus:tcp.slave1.connection=192.168.1.50
    modbus:tcp.slave1.length=10
    modbus:tcp.slave1.type=coil
 
Minimal construction in openhab.conf for serial connections will look like:

    modbus:serial.slave1.connection=/dev/ttyUSB0
    modbus:tcp.slave1.length=10
    modbus:tcp.slave1.type=coil

connects to slave at ip=192.168.1.50 and reads 10 coils starting from address 0
More complex setup could look like

    modbus:tcp.slave1.connection=192.168.1.50:502
    modbus:tcp.slave1.id=41
    modbus:poll=300
    modbus:tcp.slave1.start=0
    modbus:tcp.slave1.length=32
    modbus:tcp.slave1.type=coil

example for an moxa e1214 module in simple io mode
6 output switches starting from modbus address 0 and
6 inputs from modbus address 10000 (the function 2 implizits the modbus 10000 address range)
you only read 6 input bits and say start from 0
the moxa manual ist not right clear in this case 

    modbus:poll=300
    
    modbus:tcp.slave1.connection=192.168.6.180:502
    modbus:tcp.slave1.id=1
    modbus:tcp.slave1.start=0
    modbus:tcp.slave1.length=6
    modbus:tcp.slave1.type=coil
    
    modbus:tcp.slave2.connection=192.168.6.180:502
    modbus:tcp.slave2.id=1
    modbus:tcp.slave2.start=0
    modbus:tcp.slave2.length=6
    modbus:tcp.slave2.type=discrete
    
    modbus:tcp.slave3.connection=192.168.6.180:502
    modbus:tcp.slave3.id=1
    modbus:tcp.slave3.start=17
    modbus:tcp.slave3.length=2
    modbus:tcp.slave3.type=input
    
    modbus:tcp.slave4.connection=192.168.6.181:502
    modbus:tcp.slave4.id=1
    modbus:tcp.slave4.start=33
    modbus:tcp.slave4.length=2
    modbus:tcp.slave4.type=holding

    modbus:tcp.slave5.connection=192.168.6.181:502
    modbus:tcp.slave5.id=1
    modbus:tcp.slave5.start=10
    modbus:tcp.slave5.length=2
    modbus:tcp.slave5.type=input
    modbus:tcp.slave5.valuetype=float32

here we use the same modbus gateway with ip 192.168.6.180 twice 
on different modbus address ranges and modbus functions

NOTE: the moxa e1200 modules give by reading with function 02 from start=0 the content of register 10000 aka DI-00, an reading with function code 1 gives the address 00000 this is a little bit scary, reading from other plc can be different! 

## Item Binding Configuration

ModbusBindingProvider provides binding for Openhab Items
There are three ways to bind an item to modbus coils/registers

1) single coil/register per item

     Switch MySwitch "My Modbus Switch" (ALL) {modbus="slave1:5"}

This binds MySwitch to modbus slave defined as "slave1" in openhab.config reading/writing to the coil 5

2) separate coils for reading and writing

     Switch MySwitch "My Modbus Switch" (ALL) {modbus="slave1:<6:>7"}
In this case coil 6 is used as status coil (readonly) and commands are put to coil 7 by setting coil 7 to true.
Your hardware should then set coil 7 back to false to allow further commands processing. 

3) input coil only for reading

     Contact Contact1 "Contact1 [MAP(en.map):%s]" (All)   {modbus="slave2:0"}
In this case regarding to moxa example coil 0 is used as discrete input (in Moxa naming DI-00)

following examples are relatively useless, if you know better one let us know!
counter values in most cases 16bit values, now we must do math: in rules to deal with them ...

4) read write byte register

      Number Dimmer1 "Dimmer1 [%d]" (ALL) {modbus="slave4:0"}
and in sitemap

      Setpoint item=Dimmer1 minValue=0 maxValue=100 step=5
**NOTE:** if the value goes over a byte this case is fully untested!!!
this example should write the value to all DO bits of an moxa e1212 as byte value

5) read only byte register `type=input`

      Number MyCounterH "My Counter high [%d]" (All) {modbus="slave3:0"}
this reads counter 1 high word

      Number MyCounterL "My Counter low [%d]" (All) {modbus="slave3:1"}
this reads counter 1 low word

When using a float32 value you must use [%f] in item description.

`      Number MyCounter "My Counter [%f]" (All) {modbus="slave5:0"}`
 