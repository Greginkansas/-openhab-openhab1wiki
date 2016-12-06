_**Note:** This Binding is available in 1.9.0 and later releases._

## Table of Contents

* [Introduction](#introduction)
* [Examples](#examples)
* [Item configuration](#item-configuration)
* [Binding Configuration](#binding-configuration)
* [Change Log](#change-log)

## Introduction

The Expire binding will send a specified update or command (the "expire" update/command) to items it is bound to after a period of time has passed.  The expiration time will be started or restarted every time an update or command is received for the item that is any update or command *other than* the specified "expire" update/command.  Any future expiring update or command is cancelled if the item receives an update or command that matches the "expire" update/command.

For installation of the binding, please see the Wiki page [Bindings](Bindings), or you can add [this JAR](https://openhab.ci.cloudbees.com/job/openHAB1-Addons/lastSuccessfulBuild/artifact/bundles/binding/org.openhab.binding.expire/target/org.openhab.binding.expire-1.9.0-SNAPSHOT.jar) to your `addons` folder.

## Examples

Return a temperature sensor's state to Undefined if five minutes pass since the last numerical update:
```
Number Temperature "Temp [%.1f °C]" { expire="5m" }
```

Turn off a light one and a half hours after it was turned on:
```
Switch HallLight "HallLight [%s]" { expire="1h30m,command=OFF" }
```

Mark a motion sensor as CLOSED 30 seconds after it was opened:
```
Contact MotionSensor "MotionSensor [%s]" { expire="30s,state=CLOSED" }
```

Boil an egg for seven minutes using a Z-Wave-controlled cooker:
```
Switch EggCooker "Egg Cooker [%s]" { zwave="12", expire="7m,command=OFF" }
```

## Item Configuration

The Expire binding accepts a duration of time that can be made up of hours, minutes and seconds in the format
```
expire="1h 30m 45s"
expire="1h05s"
expire="55h 59m        12s"
```
Any part is optional, but any part present must be in the given order (hours, minutes, seconds).  Whitespace is allowed between sections.

This section can optionally be followed by a comma and the state or command to post when the timer expires.  When this optional section is not present, it defaults to posting an Undefined (`UnDefType.UNDEF`) state to the item.
```
expire="1h,command=STOP"  (send STOP command after one hour)
expire="5m,state=0"       (update state to 0 after five minutes)
expire="3m12s,Hello"      (update state to Hello after three minutes and 12 seconds)
expire="2h"               (update state to Undefined two hours after last value)
```
Note that the `state=` part is optional.

Also note that the type of item (`String`, `Number`, `Switch`, `Contact`, etc.) must accept the command or state you specify.

## Binding Configuration

The binding itself has no configuration.

## Change Log

### 1.9.0

* Initial release.
