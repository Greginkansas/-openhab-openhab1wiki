Documentation of the Modbus binding Bundle. The binding supports both TCP and Serial slaves. RTU, ASCII and BIN variants of Serial Modbus are supported.

The binding can act as 
- Modbus TCP Client (that is, as modbus master), querying data from Modbus TCP servers (that is, modbus slaves).
- Modbus serial master, querying data from modbus serial slaves

## Introduction

The Modbus TCP binding polls the slaves in an configurable interval for a configurable length. openHAB commands are translated to write requests.

For installation of the binding, please see Wiki page [[Bindings]].

 
## Details

### Supported Modbus object types
Modbus binding allows to connect to multiple Modbus slaves. The binding supports following Modbus *object types*

- coils, also known as *digital out (DO)* (read & write)
- discrete inputs, also known as *digital in (DI)* (read)
- input registers (read)
- holding registers (read & write)

Binding can be configured to interpret values stored in the 16bit registers in different ways, e.g. as signed or unsigned integer.

For more information on these object types, please consult [Modbus wikipedia article](https://en.wikipedia.org/wiki/Modbus).

### Read and write functions

Modbus specification has different operations for reading and writing different object types. These types of operations are identified by *function code*. Some devices support only certain function codes.

For more background information, please consult [Modbus wikipedia article](https://en.wikipedia.org/wiki/Modbus).

The binding uses following function codes when communicating with the slaves:

- read coils: function code (FC) 1 (*Read Coils*)
- write coil: FC 5  (*Write Single Coil*)
- read discrete inputs: FC 2 (*Read Discrete Inputs*)
- read input registers: FC 4 (*Read Input Registers*)
- read holding registers: FC 3 (*Read Multiple Holding Registers*)
- write holding register: FC 6 (*Write Single Holding Register*), OR  FC 16 (*Write Multiple Holding Registers*) (see note on `writemultipleregisters` configuration parameter below)

### Binding Configuration

add to `${openhab_home}/configuration/`

Entries in the `openhab.cfg` file should look like below.

#### Global configuration

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

#### Configuration parameters specific to each slave

The slaves are configured using key value pairs in openHAB config file using the following pattern:

     modbus:<slave-type>.<slave-name>.<slave-parameter-name>=<slave-parameter-value>

where
- `<slave-type>` can be either "tcp" or "serial" depending on the type of this Modbus slave
- `<slave-name>` is unique name per slave you are connecting to. Used in openHAB configuration to refer to the slave.
- `<slave-parameter-name>` identifies the parameter to configure
- `<slave-parameter-value>` is the value of the parameter

Valid slave parameters are

<table>
  <tr><td>parameter name</td><td>mandatory / optional?</td><td>parameter value</td></tr>
  <tr><td>connection</td><td>mandatory</td>
       <td><p>Connection string for the slave.</p>
           <p><b>TCP slaves</b> use the form <i>host_ip[:port]</i> e.g. <i>192.168.1.55</i> or <i>192.168.1.55:511</i>. If you omit port, default <i>502</i> will be used. </p>
       <p>For <b>serial connections</b> use the form <i>port:[:baud:[dataBits:[parity:[stopBits:[encoding]]]]]</i>, e.g. <i>/dev/ttyS0:38400:8:none:1:rtu</i> (read from <i>/dev/ttyS0</i> using baud rate of 38400, 8 databits, no parity, 1 stopbits, and rtu encoding. Another minimal example is <i>/dev/ttyS0:38400</i> where all optional parameters except baud rate will get their default values.</p>

<p>
<i>port</i> refers to COM port name on Windows and serial device path in *nix. Optionally one can configure one or more of the serial parameters: baud (default <i>9600</i>), dataBits (default <i>8</i>), parity (default <i>none</i>), stopBits (default <i>1</i>), encoding (default <i>ascii</i>). <br/>Options for the optional serial parameters are as follows: parity={<i>even</i>, <i>odd</i>}; encoding={<i>ascii</i>, <i>rtu</i>, <i>bin</i>}.</p>

</td>
     </tr>
  <tr><td>id</td><td>optional</td>
       <td>slave id, default 1. Also known as <i>Address</i>, <i>Station address</i>, or <i>Unit identifier</i>, see <a href="https://en.wikipedia.org/wiki/Modbus">wikipedia</a> and <a href="http://www.simplymodbus.ca/index.html">simplymodbus</a> articles for more information</td>
      </tr>

       <tr><td>type</td><td>mandatory</td>
              <td>object type. Dictates the function codes used for read & write. See below for more information. Can be either <i>coil</i>, <i>discrete</i>, <i>holding</i>, <i>input</i> or <i>register</i>.</td>
            </tr>
       
       <tr><td>start</td><td>optional</td>
              <td>address of first coil/discrete input/register to read. Default is 0. The address is directly passed to the corresponding Modbus request, and thus is zero-based. See below for more information on the addressing. </td>
            </tr>

       <tr><td>length</td><td>mandatory</td><td>number of <i>data items</i> to read. <i>Data items</i> here refers to registers, coils or discrete inputs depending on the slave type. For example, if the goal is to read one item with `valuetype=int32`, one needs to read two registers (2 * 16bit = 32bit), thus `length = 2`. If three coils are of interest, one should specify `length = 3`</td>
              </tr>
</table>
</td>

##### Comment on addressing
[Modbus Wikipedia article](https://en.wikipedia.org/wiki/Modbus#Coil.2C_discrete_input.2C_input_register.2C_holding_register_numbers_and_addresses) summarizes this excellently:

> In the traditional standard, [entity] numbers for those entities start with a digit, followed by a number of four digits in range 1–9,999:

> - coils numbers start with a zero and then span from 00001 to 09999
> - discrete input numbers start with a one and then span from 10001 to 19999
> - input register numbers start with a three and then span from 30001 to 39999
> - holding register numbers start with a four and then span from 40001 to 49999

> This translates into [entity] addresses between 0 and 9,998 in data frames.

The openhab modbus binding uses entity addresses when referring to modbus entities. That is, the entity address configured in modbus binding is passed to modbus protocol frame as-is. For example, modbus slave definition with `start=3`, `length=2` and `type=holding` will read modbus entities with the following numbers 40004 and 40005.

##### Many modbus binding slaves for single physical slave

One needs to configure as many modbus slaves to openhab as there are corresponding modbus requests. For example, in order to poll status of `coil` and `holding` items from a single [physical] modbus slave, two separate modbus slave definitions need to be configured in the ``openhab.cfg``. For example:

````
modbus:serial.slave1.connection=/dev/pts/8:38400:8:none:1:rtu
modbus:serial.slave1.type=coil
modbus:serial.slave1.length=3

modbus:serial.slave2.connection=/dev/pts/8:38400:8:none:1:rtu
modbus:serial.slave2.type=holding
modbus:serial.slave2.length=5
````

##### Read and write functions (modbus slave type)
Modbus read functions 
- `type=coil` uses function 1 "Read Coil Status" 
- `type=discrete` uses function 2 "Read Input Status" (readonly inputs)
- `type=holding` uses function 3, "Read Holding Registers"
- `type=input` uses function 4 "Read Input Register" (readonly-registers eG analogue inputs)

Modbus write functions 
- `type=coil` uses function 5 "Write Single Coil"
- `type=holding` uses function 6 "Write Single Register", or function 16 "Write Multiple registers" when `writemultipleregisters` is `true`

See also [simplymodbus.ca](http://www.simplymodbus.ca) and [wikipedia article](https://en.wikipedia.org/wiki/Modbus#Supported_function_codes).

### Config Examples

- Minimal construction in openhab.cfg for TCP connections will look like:

````
    # read 10 coils starting from address 0
    modbus:tcp.slave1.connection=192.168.1.50
    modbus:tcp.slave1.length=10
    modbus:tcp.slave1.type=coil
````
 
- Minimal construction in openhab.cfg for serial connections will look like:

````
    # read 10 coils starting from address 0
    modbus:serial.slave1.connection=/dev/ttyUSB0
    modbus:tcp.slave1.length=10
    modbus:tcp.slave1.type=coil
````

- More complex setup could look like

````
    # Poll values very 300ms = 0.3 seconds
    modbus:poll=300

    # Connect to modbus slave at 192.168.1.50, port 502
    modbus:tcp.slave1.connection=192.168.1.50:502
    # use slave id 41 in requests
    modbus:tcp.slave1.id=41
    # read 32 coils (digital outputs) starting from address 0
    modbus:tcp.slave1.start=0
    modbus:tcp.slave1.length=32
    modbus:tcp.slave1.type=coil
````

- Another example where coils, discrete inputs (`discrete`) and input registers (`input`) are polled from modbus tcp slave at `192.168.6.180`.

> (original example description:)
> example for an moxa e1214 module in simple io mode
> 6 output switches starting from modbus address 0 and
> 6 inputs from modbus address 10000 (the function 2 implizits the modbus 10000 address range)
> you only read 6 input bits and say start from 0
> the moxa manual ist not right clear in this case 

````
    modbus:poll=300
    
    # Query coils from 192.168.6.180
    modbus:tcp.slave1.connection=192.168.6.180:502
    modbus:tcp.slave1.id=1
    modbus:tcp.slave1.start=0
    modbus:tcp.slave1.length=6
    modbus:tcp.slave1.type=coil
    
    # Query discrete inputs from 192.168.6.180
    modbus:tcp.slave2.connection=192.168.6.180:502
    modbus:tcp.slave2.id=1
    modbus:tcp.slave2.start=0
    modbus:tcp.slave2.length=6
    modbus:tcp.slave2.type=discrete
    
    # Query input registers from 192.168.6.180
    modbus:tcp.slave3.connection=192.168.6.180:502
    modbus:tcp.slave3.id=1
    modbus:tcp.slave3.start=17
    modbus:tcp.slave3.length=2
    modbus:tcp.slave3.type=input
    
    # Query holding registers from 192.168.6.181
    # Holding registers matching addresses 33 and 34 are read
    modbus:tcp.slave4.connection=192.168.6.181:502
    modbus:tcp.slave4.id=1
    modbus:tcp.slave4.start=33
    modbus:tcp.slave4.length=2
    modbus:tcp.slave4.type=holding

    # Query 2 input registers from 192.168.6.181. 
    # Interpret the two registers as single 32bit floating point number
    modbus:tcp.slave5.connection=192.168.6.181:502
    modbus:tcp.slave5.id=1
    modbus:tcp.slave5.start=10
    modbus:tcp.slave5.length=2
    modbus:tcp.slave5.type=input
    modbus:tcp.slave5.valuetype=float32
````

Above we used the same modbus gateway with ip 192.168.6.180 multiple times 
on different modbus address ranges and modbus functions.

### Register interpretation (valuetype) on read & write

Note that this section applies to register elements only (`holding` or `input` type)

#### Read

When the binding interprets and converts polled input registers (`input`) or holding registers (`holding`) to openhab items, the process goes like this:

- 1. register(s) are first parsed to a number (see below for the details, exact logic depends on `valuetype`)
- 2a. if the item is Switch or Contact: zero is converted CLOSED / OFF. Other numbers are converted to OPEN / ON.
- 2b. if the item is Number: the value is passed as is

The logic for converting read registers to number goes as below. Different procedure is taken depending on `valuetype`. 

Note that <i>first register</i> refers to register with address `start` (as defined in the slave definition), <i>second register</i> refers to register with address `start + 1` etc. The <i>index</i> refers to item read index, e.g. item `Switch MySwitch "My Modbus Switch" (ALL) {modbus="slave1:5"}` has 5 as read index.

`valuetype=bit`:
- a single bit is read from the registers
- indices between 0...15 (inclusive) represent bits of the first register
- indices between 16...31 (inclusive) represent bits of the second register, etc.
- index 0 refers to the least significant bit of the first register
- index 1 refers to the second least significant bit of the first register, etc.

`valuetype=int8`:
- a byte (8 bits) from the registers is interpreted as signed integer
- index 0 refers to low byte of the first register, 1 high byte of first register
- index 2 refers to low byte of the second register, 3 high byte of second register, etc.
- it is assumed that each high and low byte is encoded in most significant bit first order

`valuetype=uint8`:
- same as INT8 except values are interpreted as unsigned integers

`valuetype=int16`:
- register with index (counting from zero) is interpreted as 16 bit signed integer.
- it is assumed that each register is encoded in most significant bit first order

`valuetype=uint16`:
- same as INT16 except values are interpreted as unsigned integers

`valuetype=int32`:
- registers (2 index) and ( 2 *index + 1) are interpreted as signed 32bit integer.
- it assumed that the first register contains the most significant 16 bits
- it is assumed that each register is encoded in most significant bit first order

`valuetype=uint32`:
- same as UINT32 except values are interpreted as unsigned integers

`valuetype=float32`:
- registers (2 index) and ( 2 *index + 1) are interpreted as signed 32bit floating point number.
- it assumed that the first register contains the most significant 16 bits
- it is assumed that each register is encoded in most significant bit first order


Extra notes
- `valuetype`s smaller than one register (less than 16 bits) actually read the whole register, and finally extract single bit from the result.
- work is ongoing (issue [#3558](https://github.com/openhab/openhab/issues/3558)) to support decoding 32bit `valuetype`s with little endian order.

#### Write

When the binding processes openhab command (e.g. sent by `sendCommand` as explained [here](https://github.com/openhab/openhab/wiki/Actions)), the process goes as follows

1. it is checked whether the associated item is bound to holding register. If not, command is ignored.
2. command is converted to 16bit integer (in [two's complement format](https://www.cs.cornell.edu/~tomf/notes/cps104/twoscomp.html)) (see below for details)
3. the 16bits are written to the register with address `start` (as defined in the slave definition)

Conversion rules for converting command to 16bit integer
- UP, ON, OPEN commands that are converter to number 1
- DOWN, OFF, CLOSED commands are converted to number 0 
- Decimal commands are truncated as 32 bit integer (in 2's complement representation), and then the least significant 16 bits of this integer are extracted.

**Note: The way Decimal commands are handled currently means that it is probably not useful to try to use Decimal commands with non-16bit `valuetype`s.**

Converting INCREASE and DECREASE commands to numbers is more complicated
1. Register matching (`start` + read index) is interpreted as unsigned 16bit integer. Previous polled register value is used
2. add/subtract `1` from the integer

**Note: note that INCREASE and DECREASE ignore valuetype when using the previously polled value. Thus, it is not recommended to use INCREASE and DECREASE commands with other than `valuetype=uint16`**

## Item Binding Configuration

ModbusBindingProvider provides binding for openHAB Items.

There are three ways to bind an item to modbus coils/registers. 

### Single coil/register per item

     Switch MySwitch "My Modbus Switch" (ALL) {modbus="slave1:5"}

- This binds MySwitch to modbus slave defined as "slave1" in openhab.cfg reading/writing to the coil (5 + slave's `start` index). The `5` is called item read index.
- If the slave is read-only, that is the `type` is `input` or `discrete`, the binding ignores any write commands. 
- if the slave1 refers to registers, and after parsing using the registers as rules defined by the `valuetype`, zero value is considered as `OFF`, everything else as `ON`.

### Separate coils for reading and writing

     Switch MySwitch "My Modbus Switch" (ALL) {modbus="slave1:<6:>7"}

- In this case coil 6 is used as status coil (read-only) and commands are put to coil 7 by setting coil 7 to true.
- (?) Your hardware should then set coil 7 back to false to allow further commands processing (Note 16.3.2016: does this relate to [issue #3685](https://github.com/openhab/openhab/issues/3685)?).

### input coil only for reading

     Contact Contact1 "Contact1 [MAP(en.map):%s]" (All)   {modbus="slave2:0"}

- In this case regarding to moxa example coil 0 is used as discrete input (in Moxa naming DI-00)
- (?) following examples are relatively useless, if you know better one let us know!
counter values in most cases 16bit values, now we must do math: in rules to deal with them ...

### Read / write register (number) 

      Number Dimmer1 "Dimmer1 [%d]" (ALL) {modbus="slave4:0"}
and in sitemap you can for example

      Setpoint item=Dimmer1 minValue=0 maxValue=100 step=5

**NOTE:** if the item value goes over the max value specified by the `valuetype` (e.g. 32767 with `int16`), the effects are fully untested!!!

(?) this example should write the value to all DO bits of an moxa e1212 as byte value

5. read only register `type=input`

      Number MyCounterH "My Counter high [%d]" (All) {modbus="slave3:0"}
this reads counter 1 high word when valuetype=`int8` or `uint8`

      Number MyCounterL "My Counter low [%d]" (All) {modbus="slave3:1"}
this reads counter 1 low word when valuetype=`int8` or `uint8`

6. floating point value numbers

When using a float32 value you must use [%f] in item description.

`      Number MyCounter "My Counter [%f]" (All) {modbus="slave5:0"}`
 