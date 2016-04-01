The page contains a number of examples for use with the [[Ecobee binding|Ecobee-Binding]] and [action](https://github.com/openhab/openhab/wiki/Actions#ecobee-actions) bundles.

## Table of Contents

* [Basic configuration](#basic-configuration)
* [Tracking last occupancy](#tracking-last-occupancy)

## Basic configuration

The following items, sitemap and rules files show the actual temperature and humidity, the thermostat's current mode and scheduled comfort setting reference.  You can set a hold for a defined comfort setting (like sleep, away, home, custom ones as well), or resume the scheduled program from all holds.  If the thermostat is in "heat" or "cool" modes, a single temperature setpoint is given to adjust the desired temperature.  If the thermostat is in "auto" mode, two temperature setpoints are given to adjust both the heat and cool setpoints.

The sitemap will also show sensor values for an ecobee3's remote sensors in the dining room, bedroom and living room.

### ecobee.items
```
DateTime lastModified "last mod [%1$tH:%1$tM:%1$tS]" { ecobee="<[123456789012#lastModified]" }
Number actualTemperature "actual temp [%.1f °F]" { ecobee="<[123456789012#runtime.actualTemperature]" }
Number actualHumidity "actual hum [%d %%]" { ecobee="<[123456789012#runtime.actualHumidity]" }
String hvacMode "hvac mode [%s]"            { ecobee="=[123456789012#settings.hvacMode]" }
String currentClimateRef "sched comf [%s]"  { ecobee="<[123456789012#program.currentClimateRef]" }
String desiredComf "desired comf"           { ecobee="<[123456789012#runningEvent.holdClimateRef]", autoupdate="false" }
Number desiredTemp "desired temp [%.1f °F]"
Number desiredHeat "desired heat [%.1f °F]" { ecobee="<[123456789012#runtime.desiredHeat]" }
Number desiredCool "desired cool [%.1f °F]" { ecobee="<[123456789012#runtime.desiredCool]" }
String desiredFan  "desired fan mode [%s]"  { ecobee="<[123456789012#runtime.desiredFanMode]" }

// Address the ecobee3 unit itself as a sensor, unlike remote sensors it also supports humidity:
Number diningRoomTemperature "dining rm temp [%.1f °F]" { ecobee="<[123456789012#remoteSensors(Dining Room).capability(temperature).value]" }
Number diningRoomHumidity "dining rm hum [%d %%]" { ecobee="<[123456789012#remoteSensors(Dining Room).capability(humidity).value]" }
Switch diningRoomOccupancy "dining rm occ [%s]" { ecobee="<[123456789012#remoteSensors(Dining Room).capability(occupancy).value]" }

Number bedroomTemperature "bedroom temp [%.1f °F]" { ecobee="<[123456789012#remoteSensors(Bedroom).capability(temperature).value]" }
Switch bedroomOccupancy "bedroom occ [%s]" { ecobee="<[123456789012#remoteSensors(Bedroom).capability(occupancy).value]" }

Number livingRoomTemperature "living rm temp [%.1f °F]" { ecobee="<[123456789012#remoteSensors(Living Room).capability(temperature).value]" }
Switch livingRoomOccupancy "living rm occ [%s]" { ecobee="<[123456789012#remoteSensors(Living Room).capability(occupancy).value]" }
```

### ecobee.sitemap
```
sitemap ecobee label="Ecobee"
{
  Frame label="Thermostat" {
    Text item=lastModified
    Text item=actualTemperature
    Text item=actualHumidity
    Switch item=hvacMode label="HVAC Mode" mappings=[heat=Heat,cool=Cool,auto=Auto,off=Off]
    Text item=currentClimateRef
    Switch item=desiredComf mappings=[sleep=Sleep,wakeup=Wake,home=Home,away=Away,smart6=Gym,resume=Resume]
    Setpoint item=desiredTemp label="Temp [%.1f °F]" minValue=50 maxValue=80 step=1 visibility=[hvacMode==heat,hvacMode==cool]
    Setpoint item=desiredHeat label="Heat [%.1f °F]" minValue=50 maxValue=80 step=1 visibility=[hvacMode==auto]
    Setpoint item=desiredCool label="Cool [%.1f °F]" minValue=50 maxValue=80 step=1 visibility=[hvacMode==auto]
    Switch item=desiredFan mappings=[on=On,auto=Auto] // for rule supported in 1.9
  }

  Frame label="Dining Room Sensors" {
    Text item=diningRoomTemperature
    Text item=diningRoomHumidity
    Text item=diningRoomOccupancy
  }

  Frame label="Bedroom Sensors" {
   Text item=bedroomTemperature
   Text item=bedroomOccupancy
  }

  Frame label="Living Room Sensors" {
   Text item=livingRoomTemperature
   Text item=livingRoomOccupancy
  }
}
```

### ecobee.rules
```
import org.openhab.core.library.types.*

rule "Populate desiredTemp from desiredHeat"
when
  Item desiredHeat received update
then
  if (hvacMode.state.toString == "heat" && desiredHeat.state instanceof DecimalType) {
    desiredTemp.postUpdate(desiredHeat.state)
  }
end

rule "Populate desiredTemp from desiredCool"
when
  Item desiredCool received update
then
  if (hvacMode.state.toString == "cool" && desiredCool.state instanceof DecimalType) {
    desiredTemp.postUpdate(desiredCool.state)
  }
end

rule TempHold
when
  Item desiredTemp received command
then
  switch hvacMode.state.toString {
    case "heat" : desiredHeat.sendCommand(receivedCommand)
    case "cool" : desiredCool.sendCommand(receivedCommand)
    case "auto" : logWarn("TempHold", "in auto mode, single setpoint ignored")
    case "off"  : logWarn("TempHold", "in off mode, single setpoint ignored")
  }
end

rule HeatHold
when
  Item desiredHeat received command
then
  logInfo("HeatHold", "Setting heat setpoint to " + receivedCommand.toString)
  val DecimalType desiredHeatTemp = receivedCommand as DecimalType
  var DecimalType desiredCoolTemp
  if (desiredCool.state instanceof DecimalType) {
    desiredCoolTemp = desiredCool.state as DecimalType
  } else {
    desiredCoolTemp = new DecimalType(90)
  }

  // the 1.9 onwards action bundle uses a selection string instead of an item as the first parameter
  // ecobeeSetHold("123456789012", desiredCoolTemp, desiredHeatTemp, null, null, null, null, null)
  // openHAB 1.8 call:
  ecobeeSetHold(desiredHeat, desiredCoolTemp, desiredHeatTemp, null, null, null, null, null)
end

rule CoolHold
when
  Item desiredCool received command
then
  logInfo("CoolHold", "Setting cool setpoint to " + receivedCommand.toString)
  val DecimalType desiredCoolTemp = receivedCommand as DecimalType
  var DecimalType desiredHeatTemp
  if (desiredHeat.state instanceof DecimalType) {
    desiredHeatTemp = desiredHeat.state as DecimalType
  } else {
    desiredHeatTemp = new DecimalType(50)
  }

  // the 1.9 onwards action bundle uses a selection string instead of an item as the first parameter
  // ecobeeSetHold("123456789012", desiredCoolTemp, desiredHeatTemp, null, null, null, null, null)
  // openHAB 1.8 call:
  ecobeeSetHold(desiredCool, desiredCoolTemp, desiredHeatTemp, null, null, null, null, null)
end

// rule supported in 1.9 onwards using the new ecobeeSetHold method signature:
rule FanHold
when
  Item desiredFan received command
then
  logInfo("FanHold", "Setting fan hold to " + receivedCommand.toString)
  val params = newLinkedHashMap(
    'isTemperatureAbsolute'-> false,
    'isTemperatureRelative' -> false,
    'isCoolOff' -> true,
    'isHeatOff' -> true,
    'coolHoldTemp' -> 90,
    'heatHoldTemp' -> 50,
    'fan' -> receivedCommand.toString)
  ecobeeSetHold("123456789012", params, null, null, null, null)
end

rule ComfortHold
when
  Item desiredComf received command
then
  if (receivedCommand.toString.equals("resume")) {
    // the 1.9 onwards action bundle uses a selection string instead of an item as the first parameter
    // ecobeeResumeProgram("123456789012", true)
    // openHAB 1.8 call:
    ecobeeResumeProgram(currentClimateRef, true)
  } else {
    // the 1.9 onwards action bundle uses a selection string instead of an item as the first parameter
    // ecobeeSetHold("123456789012", null, null, receivedCommand.toString, null, null, null, null)
    // openHAB 1.8 call:
    ecobeeSetHold(currentClimateRef, null, null, receivedCommand.toString, null, null, null, null)
  }
end
```
### Notes

1. Ecobee thermostats normally run based on a weekly schedule, but you can override the current program by setting a hold that controls the cool setpoint, the heat setpoint, and other options.  You can set a hold from a rule by calling the action `ecobeeSetHold`.  One of the parameters is a reference to a "climate" (also known as a comfort setting).  The default references for climates are `sleep`, `home`, and `away` (some models also have `wakeup`).

[Table of Contents](#table-of-contents)

## Tracking last occupancy
The ecobee3 thermostat can connect to a number of wireless remote sensors that measure occupancy and temperature.  The thermostat normally uses these to implement its "follow-me comfort" feature, where the thermostat is constantly adjusting its idea of the current ambient temperature based on an average of the temperatures of rooms that are currently occupied.

The binding can individually address each remote sensor's temperature and occupancy state.  The occupancy state is ON when there has been motion within its range in the last 30 minutes.  The rules below could be used to update a `DateTime` item to show the last time movement was sensed around any of the sensors in your home.

Items:
```
Switch EcobeeMBROccu "Ecobee MBR Occu [%s]" { ecobee="<[123456789012#remoteSensors(Bedroom).capability(occupancy).value]" }
Switch EcobeeKitchenOccu "Ecobee Kitchen Occu [%s]" { ecobee="<[123456789012#remoteSensors(Kitchen).capability(occupancy).value]" }
Switch EcobeeDROccu "Ecobee DR Occu [%s]" { ecobee="<[123456789012#remoteSensors(Dining Room).capability(occupancy).value]" }
DateTime LastOccuTime "Last Occu [%1$tm/%1$td %1$tH:%1$tM]"
```
Rules:
```
import org.openhab.core.library.types.*
import org.joda.time.*

rule LastMotionON
when
    Item EcobeeMBROccu changed to ON or
    Item EcobeeKitchenOccu changed to ON or
    Item EcobeeDROccu changed to ON
then
    postUpdate(LastOccuTime, new DateTimeType())
end

rule LastMotionOFF
when
    Item EcobeeMBROccu changed to OFF or
    Item EcobeeKitchenOccu changed to OFF or
    Item EcobeeDROccu changed to OFF
then
    switch LastOccuTime.state {
      DateTimeType: {
        var DateTime halfHourAgo = now.minusMinutes(30)
        var DateTime lastKnownMotion = new DateTime((LastOccuTime.state as DateTimeType).calendar.timeInMillis)
        if (halfHourAgo.isAfter(lastKnownMotion)) {
          LastOccuTime.postUpdate(new DateTimeType(halfHourAgo.toString))
        }
      }
    }
end
```
[Table of Contents](#table-of-contents)
