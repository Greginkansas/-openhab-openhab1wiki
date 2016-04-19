# CUL Transport
This is the common layer shared between different CUL bindings that takes care of the transport. It is of no use alone without one of the bindings making use of it.

## CUL
CUL is a collection of devices produced by busware.de which allows to interact with various protocols and devices over radio frequencies. It's also possible to build one on your own.
- Original Manufacturer: http://busware.de
- Build your own (German) http://www.fhemwiki.de/wiki/Selbstbau_CUL
- Original firmware: http://culfw.de
- Alternative firmware: https://github.com/heliflieger/a-culfw

## Installation
- drop the bundle you want to use (e.g. Intertechno bundle) into the addon folder
- drop this into the addon folder
- ensure that the user running openHAB is in the 'dialout' group or equivalent to be able to access serial ports

## Configuration

### openhab.cfg

At the very least you have to specify the device the binding should use (replace <BINDING_NAME> by the actual binding you're using, e.g. culintertechno):

#### Local
- Mandatory `<BINDING_NAME>:device=serial:/dev/ttyACM0`
- Optional `<BINDING_NAME>:baudrate=<BAUDRATE>`
where `<BAUDRATE>` is one of 75, 110, 300, 1200, 2400, 4800, 9600, 19200, 38400, 57600, 115200
- Optional `<BINDING_NAME>:parity=<PARITY>`
where `<PARITY>` is one of EVEN, ODD, MARK, NONE, SPACE

#### Network
- Mandatory `<BINDING_NAME>:device=network:192.0.0.5`

## Additional hints
It is possible that you need to explicitly specify the serial port when openhab is launched, add the following to the start script. This should normally not be needed
- -Dgnu.io.rxtx.SerialPorts=/dev/ttyACM0