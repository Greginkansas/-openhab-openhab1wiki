IPX800 is a 8 relay webserver from gce-electronics with a lot of possibility :
* 8 Digital Input
* 8 Relay (250v/ 10A / channel)
* 4 Analog Input
* 8 Counters
* Ability to cascade up to 3 extensions for a total of 32 inputs / 32 relay

Each ipx800 connected to openhab must be configured with the setting 'Send data on status changed' on the website in M2M > TCP client.

To make it simple, IPX800 is a simple device that drive output and retrieve input. On input we generally connect push button (for instance house switchs), on ouputs we can connect light bulbs for instance.



## Binding features
 * Multi ipx support
 * Direct TCP connection
 * Extensions support with alias name
 * Auto reconnect
 * Input to output redirection (even to different ipx)
 * Counter with custom average mode
 * Simple/double clic
 * Virtual dimmer
 * Pulse mode support

## Binding configuration
Add this in your openhab.cfg file
```
############################### IPX800 Binding ###################################
# Ip or hostname of ipx800
#ipx800:<name>.host=<ip or hostname>

# Tcp client connection port (optional, default to 9870)
#ipx800:<name>.port=

# Extension setup
# This step is needed to declare extensions and give them alias
# Two types of extensions are supported : x880, x400

# Ex: this declare a x880 extension connected to ipx <name> on the first address using alias <alias>
#ipx800:<name>.x880.1=<alias>
# Ex: this declare a x400 extension connected to ipx <name> on the second address using alias <alias>
#ipx800:<name>.x400.2=<alias>
```
## Items

### Syntax
ipx800 items are described as below (italic items are optionnal)

ipx800="name:port:*options>to\_name:to\_port*"
* name : ipx name or extension alias as defined in openhab.cfg
* port : ipx port name as Tnn, with T port type (O : ouput, I : input, C : Counter, A : Analog) and nn port number
* options : depending on items
* >*to\_name:*to\_port : redirection option, is used to drive directly one output using an input. to\_name is the optionnal name of the ipx800 to send to command. to\_port is the port to send the command to (if no to_name, the command will be send to the same ipx)


### Items types
#### Output
*Switch Output { ipx800="myipx:O01" }*

*Switch Output { ipx800="myipx:O01:p" }*

Drive output directly from a openhab item. Option p put this ouput in pulse mode (sending SetNxxp to ipx800)

#### Mirror
*Switch InputMirror { ipx800="myipx:I08:m" }*

State of this item will follow the input state.

#### Normal astable switch
*Switch InputNormal { ipx800="myipx:I08" }*

On each rising edge of the input, item state will change.

#### Simple Clic
*Switch InputSimpleClic { ipx800="myipx:I08:d", milight="m1;3" }*

When coupled with a double clic, after a single rising/falling edge, will wait the double clic timeout before changing item state.

#### Double clic
*Switch InputDoubleClic2 { ipx800="myipx:I08:D>myipx2:O03" }*

Change item state after a double clic on the input.

#### Virtual dimmer
*Switch InputDimmer { ipx800="ipx1:I02:v :<step>" }*

A long press will raise the value of this item each 500ms by <step>.
Once item value reach 100, it will stick to this value. A new long press will restart the dimmer to 0.

#### Simple Counter
*Number SimpleCounter {ipx800="myipx:C01"}*

Will reflect ipx800 counter value.

#### Average counter
*Number AverageCounter {ipx800="myipx\:C01\: a\:1\:m"}*

Will compute the average based on the counter. This is very useful to use in conjunction to pulse based counter (water/gaz/electrical counter). 

8 different counter could be connected direclty to ipx800.

With this kind of counter, all the power will be monitored easily (liter per minute, Kw...)

This item will publish its state at least each period.

Options : \<step\>\:\<period\>
* Step : unit of each counter increment (as defined by hardware counter)
* Period : Base period to compute the average


### Configuration example
```
Switch Output { ipx800="myipx:O01" }
Switch InputNormal { ipx800="myipx:I08" }
Switch InputSimpleClic { ipx800="myipx:I08:d>O02" }
Switch InputSimpleClic { ipx800="myipx:I08:d", milight="m1;3" }
Switch InputDoubleClic { ipx800="myipx:I08:D", milight="m1;9" }
Switch InputDoubleClic2 { ipx800="myipx:I08:D>myipx2:O03" }
Switch InputDoubleClic3 { ipx800="myipx:I08:v>O04" }
Switch Output2 { ipx800="myipx:O02" }
Switch Output3 { ipx800="myipx:O03" }
Switch Output9 { ipx800="myipx2:O01" }

Switch InputToOuput {ipx800="myipx:I08>O01"}

Switch Mirror {ipx800="myipx:I06:m>O06" }

Number PowerSimple {ipx800="myipx:C01"}
Number PowerAverage {ipx800="myipx:C01:a:1:m"}
```