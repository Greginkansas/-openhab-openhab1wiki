The page contains examples for use with the [[Modbus binding|Modbus-Binding]].

## Table of Contents

* [Rollershutter Items](#rollershutter-items)

## Rollershutter Items

Since Modbus is a low-level protocol, more abstract concepts like controlling roller shutters can be implemented in a number of different ways.  This example (offered by @ssalonen [here](https://github.com/openhab/openhab/pull/4654#issuecomment-248996109)) is one approach using openHAB rules and the standard Modbus binding.  Adapt it to your needs, and please edit this wiki page to correct and clarify this example.

### default.rules

```
import org.openhab.core.library.types.DecimalType
import org.openhab.core.library.types.PercentType
// comment out the above imports if running on openHAB 2+

rule "Process roller shutter commands"
when
    Item myRS received command
then
    logInfo("rollerShutterRules", "Processing RS command")
    if(receivedCommand != null){        
        switch(receivedCommand.toString.upperCase) {        
            case "UP" : {
                sendCommand(myRSUpDown, 1)
            }
            case "DOWN":{
                sendCommand(myRSUpDown, -1)
            }
            case "STOP":{ 
                sendCommand(myRSMoveStop, 1)
            }   
            case "MOVE":{
                sendCommand(myRSMoveStop, 0)
            }
        }
    }
end

rule "Update roller shutter position"
when
    Item myRSMoveStop received update
then
    logInfo("rollerShutterRules", "Processing position update")
    var currentPosition = myRSMoveStop.state as DecimalType
    postUpdate(myRS, new PercentType(currentPosition.toBigDecimal))
end
```

### default.items

```

// Main roller shutter item. 
// - State will be updated from register index 0. 
// - Up (1) & Down (-1) commands will be written to register index 1
// - Move (0) & Stop (1) commands will be written to register index 2
Rollershutter myRS "RollerShutter" (livingRoom)

// Helper items for writing data to MODBUS
Number myRSUpDown "RollerShutter Up/Down Control (writing to index 1, read from index 0)" (livingRoom) {modbus="slave1:<0:>1"} 
Number myRSMoveStop "RollerShutter Move/Stop Control (writing to index 2, read from index 0)" (livingRoom) {modbus="slave1:<0:>2"} 
```

### default.sitemap

```
sitemap demo label="Main Menu" {
    // https://github.com/openhab/openhab/wiki/Rollershutter-Bindings
    Switch item=myRS label="Roller shutter [(%d)]" mappings=[UP="up", STOP="X", DOWN="down"]
    Text item=myRSMoveStop
    Text item=myRSUpDown
}
```

### For local testing

Starting up modbus tcp server (i.e. modbus slave)

```
./diagslave -m tcp -p 55502
```

openhab.cfg (openHAB 1.x)

```
modbus:tcp.slave1.connection=localhost:55502:0:0:0:1
modbus:tcp.slave1.type=holding
modbus:tcp.slave1.length=20
modbus:tcp.slave1.postundefinedonreaderror=true
```

modbus.cfg (openHAB 2+)

```
tcp.slave1.connection=localhost:55502:0:0:0:1
tcp.slave1.type=holding
tcp.slave1.length=20
tcp.slave1.postundefinedonreaderror=true
```

As inspiration I used [Roller shutter wiki page](https://github.com/openhab/openhab/wiki/Rollershutter-Bindings)

[Table of Contents](#table-of-contents)

