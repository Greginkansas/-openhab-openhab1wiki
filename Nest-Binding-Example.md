## [[Nest Binding|Nest-Binding]] Example

The contents of `nest.items` and `nest.sitemap` demonstrate a possible user interface that's backed by the Nest binding.

![sample](http://watou.github.io/images/nest-binding-example.jpg)

### nest.items

The items file assumes your structure is called "Home," your thermostat is called "Dining Room," and you have two Nest Protects called "Upstairs" and "Basement."  Change these to match how your devices are named at nest.com. 
```xtend
String home_away "Home/Away [%s]" <present> { nest="=[structures(Home).away]" }
DateTime dining_room_last_connection "Last Connection [%1$tm/%1$td/%1$tY %1$tH:%1$tM:%1$tS]" <calendar> {nest="<[thermostats(Dining Room).last_connection]"}
String dining_room_hvac_mode "HVAC Mode" <climate> { nest="=[thermostats(Dining Room).hvac_mode]" }
String dining_room_temperature_scale "Temperature Scale [%s]" { nest="<[thermostats(Dining Room).temperature_scale]" }
Number dining_room_ambient_temperature_f "Ambient Temperature [%.1f °F]" <temperature> { nest="<[thermostats(Dining Room).ambient_temperature_f]" }
Number dining_room_ambient_temperature_c "Ambient Temperature [%.1f °C]" <temperature> { nest="<[thermostats(Dining Room).ambient_temperature_c]" }
Number dining_room_humidity "Humidity [%d %%]" <humidity> { nest="<[thermostats(Dining Room).humidity]" }
Number dining_room_target_temperature_f "Target Temperature [%.1f °F]" <temperature> { nest="=[thermostats(Dining Room).target_temperature_f]" }
Number dining_room_target_temperature_low_f "Minimum Temperature [%.1f °F]" <temperature> { nest="=[thermostats(Dining Room).target_temperature_low_f]" }
Number dining_room_target_temperature_high_f "Maximum Temperature [%.1f °F]" <temperature> { nest="=[thermostats(Dining Room).target_temperature_high_f]" }
Number dining_room_away_temperature_low_f "Minimum Away Temp. [%.1f °F]" <temperature> { nest="<[thermostats(Dining Room).away_temperature_low_f]" }
Number dining_room_away_temperature_high_f "Maximum Away Temp. [%.1f °F]" <temperature> { nest="<[thermostats(Dining Room).away_temperature_high_f]" }
String basement_smoke "Smoke Status [%s]" <smoke> { nest="<[smoke_co_alarms(Basement).smoke_alarm_state]" }
String basement_co "CO Status [%s]" <co> { nest="<[smoke_co_alarms(Basement).co_alarm_state]" }
String basement_state "Status Color [%s]" { nest="<[smoke_co_alarms(Basement).ui_color_state]" }
DateTime basement_last_connection "Last Connection [%1$tm/%1$td/%1$tY %1$tH:%1$tM:%1$tS]" <calendar> {nest="<[smoke_co_alarms(Basement).last_connection]"}
String upstairs_smoke "Smoke Status [%s]" <smoke> { nest="<[smoke_co_alarms(Upstairs).smoke_alarm_state]" }
String upstairs_co "CO Status [%s]" <co> { nest="<[smoke_co_alarms(Upstairs).co_alarm_state]" }
String upstairs_state "Status Color [%s]" { nest="<[smoke_co_alarms(Upstairs).ui_color_state]" }
DateTime upstairs_last_connection "Last Connection [%1$tm/%1$td/%1$tY %1$tH:%1$tM:%1$tS]" <calendar> {nest="<[smoke_co_alarms(Upstairs).last_connection]"}
```

### nest.sitemap

The sitemap will show the ambient temperature in the same scale (C or F) that the thermostat is set to.  It will only show the away temperature range if the structure is currently in away mode.  It will show the single setpoint if the thermostat is either in "heat" or "cool" mode, or will show the low and high setpoints if the thermostat is in "heat-cool" (auto) mode.  The smoke+CO detectors show their status colors in the same color as the Nest Protect's light ring.
```xtend
sitemap nest label="Nest"
{
  Frame label="Home" icon="house" {
    Switch item=home_away mappings=[home="Home",away="Away"]
  }
  Frame label="Dining Room Thermostat" {
    Text item=dining_room_ambient_temperature_f visibility=[dining_room_temperature_scale=="F"]
    Text item=dining_room_ambient_temperature_c visibility=[dining_room_temperature_scale=="C"]
    Text item=dining_room_humidity
    Switch item=dining_room_hvac_mode mappings=[heat="Heat",cool="Cool","heat-cool"="Auto",off="Off"]
    Text item=dining_room_away_temperature_low_f visibility=[home_away=="away",home_away=="auto-away"]
    Text item=dining_room_away_temperature_high_f visibility=[home_away=="away",home_away=="auto-away"]
    Setpoint item=dining_room_target_temperature_f label="Target Temperature [%.1f °F]" minValue=50 maxValue=80 step=1 visibility=[dining_room_hvac_mode=="heat",dining_room_hvac_mode=="cool"]
    Setpoint item=dining_room_target_temperature_low_f label="Minimum Temperature [%.1f °F]" minValue=50 maxValue=80 step=1 visibility=[dining_room_hvac_mode=="heat-cool"]
    Setpoint item=dining_room_target_temperature_high_f label="Maximum Temperature [%.1f °F]" minValue=50 maxValue=80 step=1 visibility=[dining_room_hvac_mode=="heat-cool"]
    Text item=dining_room_last_connection
  }
  Frame label="Basement Nest Protect" {
   Text item=basement_smoke valuecolor=[basement_state=="green"="green",basement_state=="gray"="gray",basement_state=="yellow"="yellow",basement_state=="red"="red"]
   Text item=basement_co valuecolor=[basement_state=="green"="green",basement_state=="gray"="gray",basement_state=="yellow"="yellow",basement_state=="red"="red"]
   Text item=basement_last_connection
  }
  Frame label="Upstairs Nest Protect" {
   Text item=upstairs_smoke valuecolor=[upstairs_state=="green"="green",upstairs_state=="gray"="gray",upstairs_state=="yellow"="yellow",upstairs_state=="red"="red"]
   Text item=upstairs_co valuecolor=[upstairs_state=="green"="green",upstairs_state=="gray"="gray",upstairs_state=="yellow"="yellow",upstairs_state=="red"="red"]
   Text item=upstairs_last_connection
  }
}
```

### ${openhabhome}/webapps/images/

I also created co*.png, smoke*.png, humidity.png and other icons by copying existing icons:
```shell
cp shield.png co.png
cp shield-1.png co-ok.png
cp shield-0.png co-warning.png
cp shield-0.png co-emergency.png
cp shield.png smoke.png
cp shield-1.png smoke-ok.png
cp shield-0.png smoke-warning.png
cp shield-0.png smoke-emergency.png
cp softener.png humidity.png
cp present.png present-home.png
cp present-off.png present-away.png
cp present-off.png present-auto-away.png
```

### Rules

#### Opening Windows, set the Thermostat to away-mode to save Energy
This rule assumes that the windows are all `Contact` Items, are all in a Group called `GWindow`, and that the members are `Bedroom2ZoneTripped` ... `StairsWindowsZoneTripped` per the list below.

```xtend
rule "Windows Opened (any)"
  when
    Item Bedroom2ZoneTripped changed from CLOSED to OPEN or
    Item Bedroom3ZoneTripped changed from CLOSED to OPEN or
    Item FamilyRoomZoneTripped changed from CLOSED to OPEN or
    Item GuestBathZoneTripped changed from CLOSED to OPEN or
    Item KitchenZoneTripped changed from CLOSED to OPEN or
    Item LivingRoomZoneTripped changed from CLOSED to OPEN or
    Item MasterBath1ZoneTripped changed from CLOSED to OPEN or
    Item MasterBath2ZoneTripped changed from CLOSED to OPEN or
    Item MasterBath3ZoneTripped changed from CLOSED to OPEN or
    Item MasterBedroomZoneTripped changed from CLOSED to OPEN or
    Item StairsWindowsZoneTripped changed from CLOSED to OPEN
  then
    if (GWindow.members.filter(s|s.state==OPEN).size == 1) {
      home_away.sendCommand("away")
    }
end
```

#### Closing Windows, set the Thermostat to home-mode when all windows are closed

As the counterpart of the above rule, set the Thermostat to `home` once all the windows are closed.

```xtend
rule "Windows Closed (all)"
  when
    Item Bedroom2ZoneTripped changed from OPEN to CLOSED or
    Item Bedroom3ZoneTripped changed from OPEN to CLOSED or
    Item FamilyRoomZoneTripped changed from OPEN to CLOSED or
    Item GuestBathZoneTripped changed from OPEN to CLOSED or
    Item KitchenZoneTripped changed from OPEN to CLOSED or
    Item LivingRoomZoneTripped changed from OPEN to CLOSED or
    Item MasterBath1ZoneTripped changed from OPEN to CLOSED or
    Item MasterBath2ZoneTripped changed from OPEN to CLOSED or
    Item MasterBath3ZoneTripped changed from OPEN to CLOSED or
    Item MasterBedroomZoneTripped changed from OPEN to CLOSED or
    Item StairsWindowsZoneTripped changed from OPEN to CLOSED
  then
    if (GWindow.members.filter(s|s.state==OPEN).size == 0) {
      home_away.sendCommand("home")
    }
end
```
