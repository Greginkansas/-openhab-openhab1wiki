Documentation of the Modbus binding Bundle. The binding supports both TCP and Serial slaves. RTU, ASCII and BIN variants of Serial Modbus are supported.

The binding acts as Modbus TCP Client (that is, as modbus master), querying data from Modbus TCP servers (that is, modbus slaves).

## Introduction

The Modbus TCP binding polls the slaves in an configurable interval for a configurable length. openHAB commands are translated to write requests.

For installation of the binding, please see Wiki page [[Bindings]].

 
## Details

## Supported Modbus object types
Modbus binding allows to connect to multiple Modbus slaves. The binding supports following Modbus *object types*

- coils, also known as *digital out (DO)* (read & write)
- discrete inputs, also known as *digital in (DI)* (read)
- input registers (read)
- holding registers (read & write)

Binding can be configured to interpret values stored in the 16bit registers in different ways, e.g. as signed or unsigned integer.

For more information on these object types, please consult [Modbus wikipedia article](https://en.wikipedia.org/wiki/Modbus).

## Read and write functions

Modbus specification has different operations for reading and writing different object types. These types of operations are identified by *function code*. Some devices support only certain function codes.

For more background information, please consult [Modbus wikipedia article](https://en.wikipedia.org/wiki/Modbus).

The binding uses following function codes when communicating with the slaves:

- read coils: function code (FC) 1 (*Read Coils*)
- write coil: FC 5  (*Write Single Coil*)
- read discrete inputs: FC 2 (*Read Discrete Inputs*)
- read input registers: FC 4 (*Read Input Registers*)
- read holding registers: FC 3 (*Read Multiple Holding Registers*)
- write holding register: FC 6 (*Write Single Holding Register*), OR  FC 16 (*Write Multiple Holding Registers*) (see note on `writemultipleregisters` configuration parameter below)

## Binding Configuration

add to `${openhab_home}/configuration/`

Entries in the `openhab.cfg` file should look like below.

### Global configuration

Most of config parameters are related to specific slaves, but some are global and thus affect all slaves.

**Poll period (optional)**

Frequency of polling Modbus slaves can be configured using `poll` keyword:
````
     modbus:poll=<value>
````
This is optional and default is `200`. Note that the value is in milliseconds! For example, `modbus:poll=1000` makes the binding poll Modbus slaves once per second.

**Function code to use when writing holding registers (optional)**

Binding can be configured to use FC 16 (*Write Multiple Holding Registers*) over FC 6 (*Write Single Holding Register*) when writing holding register items (see above)
````
     modbus:writemultipleregisters=<value>
````
This is optional and default is `false`. For example, `modbus:writemultipleregisters=true` makes the binding to use FC16 when writing holding registers.

### Configuration parameters specific to each slave

The slaves are configured using key value pairs in openHAB config file using the following pattern:

     modbus:<slave-type>.<slave-name>.<slave-parameter-name>=<slave-parameter-value>

where
- `<slave-type>` can be either "tcp" or "serial" depending on the type of this Modbus slave
- `<slave-name>` is unique name per slave you are connecting to. Used in openHAB configuration to refer to the slave.
- `<slave-parameter-name>` identifies the parameter to configure
- `<slave-parameter-value>` is the value of the parameter

Valid slave parameters are

<table>
  <tr><td>parameter name</td><td>mandatory / optional?</td><td>parameter value</td><td>example values</td></tr>
  <tr><td>connection</td><td>mandatory</td>
       <td>connection string for the slave. TCP slaves use the form host_ip[:port] e.g. 192.168.1.55 or 192.168.1.55:511. If you omit port, default 502 will be used. <br />For serial connections use the COM port name and optionally one can configure one or more of the serial parameters. [:baud:dataBits:parity:stopBits:encoding] <br/>options are: parity={even,odd}; encoding={ascii,rtu,bin} (default is ascii, supported since v1.7)</td>
       <td><pre>192.168.1.50</pre>, <pre>/dev/ttyS0:38400:8:none:1:rtu</pre>
</td></tr>
  <tr><td>id</td><td>optional</td>
       <td>slave id, default 1. Also known as <i>Address</i>, <i>Station address</i>, or <i>Unit identifier</i>, see <a href="https://en.wikipedia.org/wiki/Modbus">wikipedia</a> and <a href="http://www.simplymodbus.ca/index.html">simplymodbus</a> articles for more information</td>
       <td></td></tr>

       <tr><td>type</td><td>mandatory</td>
              <td>object type, can be either "coil", "discrete", "holding", "input" or "register", now only "coil", "discrete", "holding" and "input" are supported</td>
              <td></td></tr>
       
       <tr><td>start</td><td>optional</td>
              <td>address of first coil/discrete input/register to read. Default is 0. The address is directly passed to the corresponding Modbus request, and thus is zero-based. </td>
              <td></td></tr>

       <tr><td>length</td><td>mandatory</td><td>number of data items to read, default 0 (but set it to something meaningful :)</td>
              <td></td></tr>
</table>

Remark : in "`openhab_default.cfg`", the modbus binding section has a wrong key "`host`", this doesn't work if you put your slave ip address here. So you have to replace "`host`" by "`connection`" which is the right key as mentioned above.

TODO: mention about `start` addressing (ie. relative to 00001, 10001, 30001, or 40001 depending on object type)
TODO: comment that one physical slave might require many slave definition in openHAB (one for each object type for example, or different slave ids)

Modbus read functions 
- `type=coil` uses function 1 "Read Coil Status" 
- `type=discrete` uses function 2 "Read Input Status" (readonly inputs)
- `type=holding` uses function 3, "Read Holding Registers"
- `type=input` uses function 4 "Read Input Register" (readonly-registers eG analogue inputs)

Modbus write functions 
- `type=coil` uses function 5 "Write Single Coil"
- `type=holding` uses function 6 "Write Single Register"
 see also http://www.simplymodbus.ca

with `type=holding` and `type=input` you can now only operate with datatype byte!!!
see point 4 below

Minimal construction in openhab.cfg for TCP connections will look like:

    modbus:tcp.slave1.connection=192.168.1.50
    modbus:tcp.slave1.length=10
    modbus:tcp.slave1.type=coil
 
Minimal construction in openhab.cfg for serial connections will look like:

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

ModbusBindingProvider provides binding for openHAB Items
There are three ways to bind an item to modbus coils/registers

1) single coil/register per item

     Switch MySwitch "My Modbus Switch" (ALL) {modbus="slave1:5"}

This binds MySwitch to modbus slave defined as "slave1" in openhab.cfg reading/writing to the coil 5

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
 