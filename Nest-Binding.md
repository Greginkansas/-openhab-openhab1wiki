This wiki will contain configuration information for the Nest binding I'm developing.  

TBD

## openhab.cfg ##

Your `openhab.cfg` file should contain these keys.

### nest:refresh ###

How often, in milliseconds, to update states.  Don't do it too frequently or you will hit API limits.

```
nest:refresh=60000
```

### nest:client_id ###
### nest:client_secret ###
### nest:pincode ###

You will have to register as a [Nest Developer](https://nest.com/developer/) and [Register a new client](https://developer.nest.com/clients/new).  Make sure to grant all the permissions you intend to use.

Once you've created your [client](https://developer.nest.com/clients), paste the Authorization URL into a new tab in your browser.  This will have you login to your normal Nest account, and will then present the pincode.

Paste all three of these values into your openhab.cfg file like so (using your actual values):

```
nest:client_id=e5cc5558-ec55-4c55-8555-4b95555f4979
nest:client_secret=ZZo28toiuoiurok4WjUya1Bnc
nest:pincode=2JTXXXJL
```

Multiple instance support (allowing the binding to access multiple Nest accounts at once) conflicts with Prohibition 3 of the [Nest Developer Terms of Service](https://developer.nest.com/documentation/cloud/tos), and so is not implemented.

## your.items file: ##

The Nest binding will support binding strings in this format:

```
Number therm_humidity "humidity [%d %%]"                { nest="<[thermostats(Name).humidity]" }
String therm_locale "locale [%s]"                       { nest="<[thermostats(Name).locale]" }
String therm_temperature_scale "temperature_scale [%s]" { nest="<[thermostats(Name).temperature_scale]" }
Switch therm_is_using_emergency_heat "is_using_emergency_heat [%s]" { nest="<[thermostats(Name).is_using_emergency_heat]" }
String therm_has_fan "has_fan [%s]"                     { nest="<[thermostats(Name).has_fan]" }
String therm_software_version "software_version [%s]"   { nest="<[thermostats(Name).software_version]" }
String therm_has_leaf "has_leaf [%s]"                   { nest="<[thermostats(Name).has_leaf]" }
String therm_device_id "device_id [%s]"                 { nest="<[thermostats(Name).device_id]" }
String therm_name "name [%s]"                           { nest="<[thermostats(Name).name]" }
String therm_can_heat "can_heat [%s]"                   { nest="<[thermostats(Name).can_heat]" }
String therm_can_cool "can_cool [%s]"                   { nest="<[thermostats(Name).can_cool]" }
String therm_hvac_mode "hvac_mode [%s]"                 { nest="<[thermostats(Name).hvac_mode]" }
Number therm_target_temperature_c "target_temperature_c [%.1f °C]"           { nest="<[thermostats(Name).target_temperature_c]" }
Number therm_target_temperature_f "target_temperature_f [%.1f °F]"           { nest="<[thermostats(Name).target_temperature_f]" }
Number therm_target_temperature_high_c "target_temperature_high_c [%.1f °C]" { nest="<[thermostats(Name).target_temperature_high_c]" }
Number therm_target_temperature_high_f "target_temperature_high_f [%.1f °F]" { nest="<[thermostats(Name).target_temperature_high_f]" }
Number therm_target_temperature_low_c "target_temperature_low_c [%.1f °C]"   { nest="<[thermostats(Name).target_temperature_low_c]" }
Number therm_target_temperature_low_f "target_temperature_low_f [%.1f °F]"   { nest="<[thermostats(Name).target_temperature_low_f]" }
Number therm_ambient_temperature_c "ambient_temperature_c [%.1f °C]"         { nest="<[thermostats(Name).ambient_temperature_c]" }
Number therm_ambient_temperature_f "ambient_temperature_f [%.1f °F]"         { nest="<[thermostats(Name).ambient_temperature_f]" }
Number therm_away_temperature_high_c "away_temperature_high_c [%.1f °C]"     { nest="<[thermostats(Name).away_temperature_high_c]" }
Number therm_away_temperature_high_f "away_temperature_high_f [%.1f °F]"     { nest="<[thermostats(Name).away_temperature_high_f]" }
Number therm_away_temperature_low_c "away_temperature_low_c [%.1f °C]"       { nest="<[thermostats(Name).away_temperature_low_c]" }
Number therm_away_temperature_low_f "away_temperature_low_f [%.1f °F]"       { nest="<[thermostats(Name).away_temperature_low_f]" }
String therm_structure_id "structure_id [%s]"           { nest="<[thermostats(Name).structure_id]" }
String therm_fan_timer_active "fan_timer_active [%s]"   { nest="<[thermostats(Name).fan_timer_active]" }
String therm_name_long "name_long [%s]"                 { nest="<[thermostats(Name).name_long]" }
String therm_is_online "is_online [%s]"                 { nest="<[thermostats(Name).is_online]" }
DateTime therm_last_connection "last_connection [%1$tm/%1$td/%1$tY %1$tH:%1$tM:%1$tS]" { nest="<[thermostats(Name).last_connection]" }

String smoke_name "name [%s]"                           { nest="<[smoke_co_alarms(Name).name]" }
String smoke_locale "locale [%s]"                       { nest="<[smoke_co_alarms(Name).locale]" }
String smoke_structure_id "structure_id [%s]"           { nest="<[smoke_co_alarms(Name).structure_id]" }
String smoke_software_version "software_version [%s]"   { nest="<[smoke_co_alarms(Name).software_version]" }
String smoke_device_id "device_id [%s]"                 { nest="<[smoke_co_alarms(Name).device_id]" }
String smoke_name_long "name_long [%s]"                 { nest="<[smoke_co_alarms(Name).name_long]" }
String smoke_is_online "is_online [%s]"                 { nest="<[smoke_co_alarms(Name).is_online]" }
DateTime smoke_last_connection "last_connection [%1$tm/%1$td/%1$tY %1$tH:%1$tM:%1$tS]" { nest="<[smoke_co_alarms(Name).last_connection]" }
String smoke_battery_health "battery_health [%s]"       { nest="<[smoke_co_alarms(Name).battery_health]" }
String smoke_smoke_alarm_state "smoke_alarm_state [%s]" { nest="<[smoke_co_alarms(Name).co_alarm_state]" }
String smoke_co_alarm_state "co_alarm_state [%s]"       { nest="<[smoke_co_alarms(Name).smoke_co_state]" }
String smoke_ui_color_state "ui_color_state [%s]"       { nest="<[smoke_co_alarms(Name).ui_color_state]" }
String smoke_is_manual_test_active "is_manual_test_active [%s]" { nest="<[smoke_co_alarms(Name).is_manual_test_active]" }
DateTime smoke_last_manual_test_time "last_manual_test_time [%1$tm/%1$td/%1$tY %1$tH:%1$tM:%1$tS]" { nest="<[smoke_co_alarms(Name).last_manual_test_time]" }

String struct_name "name [%s]"                 { nest="<[structures(Name).name]" }
String struct_country_code "country_code [%s]" { nest="<[structures(Name).country_code]" }
String struct_postal_code "postal_code [%s]"   { nest="<[structures(Name).postal_code]" }
String struct_time_zone "time_zone [%s]"       { nest="<[structures(Name).time_zone]" }
String struct_away "away [%s]"                 { nest="<[structures(Name).away]" }
String struct_structure_id "structure_id [%s]" { nest="<[structures(Name).structure_id]" }
NOT A REAL ITEM { nest="<[structures(Name).smoke_co_alarms(Name).SEE_ABOVE]" }
NOT A REAL ITEM { nest="<[structures(Name).thermostats(Name).SEE_ABOVE]" }
```

TBD