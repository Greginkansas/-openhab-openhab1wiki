## Table of Contents

* [Introduction](#introduction)
* [Binding Configuration](#binding-configuration)
* [Item configuration](#item-configuration)
 * [Handling special characters](#handling-special-characters)
* [Binding Examples](#binding-examples)
* [Logging](#logging)
* [Known Issues](#known-issues)
* [Change Log](#change-log)

## Introduction

[Nest Labs](https://nest.com/) developed/acquired the Wi-Fi enabled Nest Learning Thermostat, the Nest Protect Smoke+CO detector, and the Nest Cam.  These devices are supported by this binding, which communicates with the Nest API over a secure, RESTful API to Nest's servers. Monitoring ambient temperature and humidity, changing HVAC mode, changing heat or cool setpoints, monitoring and changing your "home/away" status, and monitoring your Nest Protects and Nest Cams can be accomplished through this binding.

> For installation of the binding JAR on your system, please see the Wiki page [Bindings](Bindings).  You can locate the nightly build of this binding [here](https://openhab.ci.cloudbees.com/job/openHAB1-Addons/lastSuccessfulBuild/artifact/bundles/binding/org.openhab.binding.nest/target/org.openhab.binding.nest-1.9.0-SNAPSHOT.jar).

## Binding Configuration

In order to use this binding, you will have to register as a [Nest Developer](https://developer.nest.com/) and [register a new Product](https://developer.nest.com/products/new).

> Make sure to grant [all the permissions](https://developers.nest.com/documentation/cloud/permissions-overview#available-permissions) you intend to use.  **When in doubt, enable the permission,** because you will otherwise have to reauthorize the binding if you later have to change the permissions.  

Leave the **Redirect URI** field **blank** for PIN-based authorization. At this point, you will have your `nest:client_id` (**Product ID**) and `nest:client_secret` (**Product Secret**).

Once you've created your [product](https://developer.nest.com/products) as above, paste the **Authorization URL** into a new tab in your browser.  This will have you login to your normal Nest account, and will then present the `nest:pin_code`.  Prepare to copy and paste your values for `nest:client_id`, `nest:client_secret` and `nest:pin_code` in order to configure the binding.

*openHAB 1.x:* Edit the file `openhab.cfg` located in `${openhab_home}/configurations/`.  Paste all three of these values into your `openhab.cfg` file like so (using _your_ values):

    ############################## Nest binding ########################################
    #
    # Data refresh interval in ms (optional, defaults to 60000)
    #nest:refresh=60000

    # HTTP request timeout in ms (as of 1.8, optional, defaults to 10000)
    #nest:timeout=10000

    # the Product ID for the product you created (required, replace with your own)
    nest:client_id=e5cc5558-ec55-4c55-8555-4b95555f4979

    # the Product Secret for the product you created (required, replace with your own)
    nest:client_secret=ZZo28toiuoiurok4WjUya1Bnc

    # the PIN code that was generated when you authorized your account to allow
    # this product (required, replace with your own)
    nest:pin_code=2JTXXXJL

An optional _refresh interval_ setting may also be specified, via the `nest:refresh` parameter, and defaults to a polling rate of one call per every 60000ms (one minute).

:warning: Setting the _refresh interval_ aggressively may cause you to hit [data rate limits](https://developer.nest.com/documentation/cloud/data-rate-limits).  Nest documentation recommends the `nest:refresh` not be set lower than 60000.

 > To avoid errors, we recommend you limit requests to one call per minute, maximum.

    nest:refresh=60000

## Item configuration

In order to bind an Item to a supported Nest product's properties, you need to provide configuration settings by adding some binding information in your .item file (in  `conf[igurations]/items/`). The syntax for the Nest binding configuration string is explained below.

Nest bindings start with a `<`, `>` or `=`, to indicate if the Item receives values from the API (in binding), sends values to the API (out binding), or both (bidirectional binding), respectively.

The first character is then followed by a section between square brackets (\[and \] characters):

```
<[<property>]
```

where `<property>` is one of a long list of properties than you can read and optionally change. See the list below, and peruse the [Nest API Reference](https://developer.nest.com/documentation/api-reference) for all specifics as to their meanings.

Since device and structure identifiers are very long, cryptic strings that are hard to learn, binding configurations allow you to use the device's textual name as a reference.  Whatever name you see in the web or mobile client is the name you would supply in an Item's binding configuration.  So, for example, in order to determine the current humidity detected at the thermostat named 'Living Room', your binding configuration would look like this:

```
Number NestTStat_humidity "Humidity [%d %%]" {nest="<[thermostats(Living Room).humidity]"}
```

### Handling special characters

With the convenience of using simple names for structures, thermostats and smoke+CO detectors, comes the price of having to handle special characters in the names.  Any characters in a name that could interfere with the parsing of the binding configuration string need to be either 1) removed from the device name in your account at nest.com, or 2) replaced with "URL-encoded" versions.  The characters that have to be replaced are `[`, `]`, `(`, `)`, `,`, `.` and `+`.  Here are some examples:

What you see at nest.com | The escaped version to use in the Nest binding
-------------------------------------|-----------------------------------------------
Dining Room (Ground Floor) | Dining Room %28Ground Floor%29
Den Smoke+CO | Den Smoke%2BCO
123 Main St. | 123 Main St%2E
Bogota, Colombia | Bogota%2C Colombia
[Basement] | %5BBasement%5D

To reiterate, you could change the display names of your devices at nest.com, and thereby avoid having to put escaped versions in your binding configuration strings.

In order to change the current HVAC mode of the Living room thermostat between `off`, `eco`, `heat`, `cool` and `heat-cool`, your item would look like:

```
String NestTStat_hvac_mode "HVAC Mode [%s]" {nest="=[thermostats(Living Room).hvac_mode]"}
```

When you update or send a command to the item with one of the five possible valid strings, you will change the HVAC mode.

Below are some examples of valid binding configuration strings, as you would define in the your .items file.  The examples represent the current set of available properties, and for each property, the example shows if it is an in-binding (`<`) only (read-only), an out-binding (`>`) only (write-only), or a bidirectional (`=`) (read/write) binding only.  Note, however, that if a read/write property is only authorized for read-only access in the Nest Product you authorized, an attempt to change its value will fail.

In this example, there is a Nest structure called `Home`, a Thermostat called `Upstairs`, a Smoke/CO Sensor called `Master Bedroom` and a Nest Cam called `Dining Room`.

```
/* Nest binding Items */

DateTime Nest_last_connection "Last Nest Connection [%1$tm/%1$td %1$tH:%1$tM]" {nest="<[last_connection]"}

/* Structures - change Home to your structure's name */

String   NestHome_name "Name [%s]"                    {nest="=[structures(Home).name]"} // writeable as of 1.9
String   NestHome_country_code "Country Code [%s]"    {nest="<[structures(Home).country_code]"}
String   NestHome_postal_code "Postal Code [%s]"      {nest="<[structures(Home).postal_code]"}
String   NestHome_time_zone "Time Zone [%s]"          {nest="<[structures(Home).time_zone]"}
String   NestHome_away "Away [%s]"                    {nest="=[structures(Home).away]"}
String   NestHome_structure_id "Structure Id [%s]"    {nest="<[structures(Home).structure_id]"}
Switch   NestHome_rhr_enrollment "Rush Hour Rewards Enrollment [%s]" {nest="<[structures(Home).rhr_enrollment]"} // as of 1.9
String   NestHome_co_alarm_state "Home CO [%s]"       {nest="<[structures(Home).co_alarm_state]"} // as of 1.9
String   NestHome_smoke_alarm_state "Home Smoke [%s]" {nest="<[structures(Home).smoke_alarm_state]"} // as of 1.9

/* Thermostats - change Upstairs to your thermostat's name */

Number   NestTStat_humidity "Humidity [%d %%]"                                   {nest="<[thermostats(Upstairs).humidity]"}
String   NestTStat_locale "Locale [%s]"                                          {nest="<[thermostats(Upstairs).locale]"}
String   NestTStat_temperature_scale "Temperature Scale [%s]"                    {nest="=[thermostats(Upstairs).temperature_scale]"} // writeable as of 1.9
Switch   NestTStat_is_using_emergency_heat "Is using emergency heat [%s]"        {nest="<[thermostats(Upstairs).is_using_emergency_heat]"}
Switch   NestTStat_has_fan "Has Fan [%s]"                                        {nest="<[thermostats(Upstairs).has_fan]"}
String   NestTStat_software_version "Software Version [%s]"                      {nest="<[thermostats(Upstairs).software_version]"}
Switch   NestTStat_has_leaf "Has Leaf [%s]"                                      {nest="<[thermostats(Upstairs).has_leaf]"}
String   NestTStat_device_id "Device Id [%s]"                                    {nest="<[thermostats(Upstairs).device_id]"}
String   NestTStat_name "Name [%s]"                                              {nest="<[thermostats(Upstairs).name]"}
Switch   NestTStat_can_heat "Can Heat [%s]"                                      {nest="<[thermostats(Upstairs).can_heat]"}
Switch   NestTStat_can_cool "Can Cool [%s]"                                      {nest="<[thermostats(Upstairs).can_cool]"}
String   NestTStat_hvac_mode "HVAC Mode [%s]"                                    {nest="=[thermostats(Upstairs).hvac_mode]"}
Number   NestTStat_target_temperature_c "Target Temperature [%.1f °C]"           {nest="=[thermostats(Upstairs).target_temperature_c]"}
Number   NestTStat_target_temperature_f "Target Temperature [%.1f °F]"           {nest="=[thermostats(Upstairs).target_temperature_f]"}
Number   NestTStat_target_temperature_high_c "Target Temperature High [%.1f °C]" {nest="=[thermostats(Upstairs).target_temperature_high_c]"}
Number   NestTStat_target_temperature_high_f "Target Temperature High [%.1f °F]" {nest="=[thermostats(Upstairs).target_temperature_high_f]"}
Number   NestTStat_target_temperature_low_c "Target Temperature Low [%.1f °C]"   {nest="=[thermostats(Upstairs).target_temperature_low_c]"}
Number   NestTStat_target_temperature_low_f "Target Temperature Low [%.1f °F]"   {nest="=[thermostats(Upstairs).target_temperature_low_f]"}
Number   NestTStat_ambient_temperature_c "Ambient Temperature [%.1f °C]"         {nest="<[thermostats(Upstairs).ambient_temperature_c]"}
Number   NestTStat_ambient_temperature_f "Ambient Temperature [%.1f °F]"         {nest="<[thermostats(Upstairs).ambient_temperature_f]"}
Number   NestTStat_eco_temperature_high_c "Eco Temperature High [%.1f °C]"       {nest="<[thermostats(Upstairs).eco_temperature_high_c]"}  // as of 1.9
Number   NestTStat_eco_temperature_high_f "Eco Temperature High [%.1f °F]"       {nest="<[thermostats(Upstairs).eco_temperature_high_f]"}  // as of 1.9
Number   NestTStat_eco_temperature_low_c "Eco Temperature Low [%.1f °C]"         {nest="<[thermostats(Upstairs).eco_temperature_low_c]"}  // as of 1.9
Number   NestTStat_eco_temperature_low_f "Eco Temperature Low [%.1f °F]"         {nest="<[thermostats(Upstairs).eco_temperature_low_f]"}  // as of 1.9
Number   NestTStat_away_temperature_high_c "Away Temperature High [%.1f °C]"     {nest="<[thermostats(Upstairs).away_temperature_high_c]"} // deprecated
Number   NestTStat_away_temperature_high_f "Away Temperature High [%.1f °F]"     {nest="<[thermostats(Upstairs).away_temperature_high_f]"} // deprecated
Number   NestTStat_away_temperature_low_c "Away Temperature Low [%.1f °C]"       {nest="<[thermostats(Upstairs).away_temperature_low_c]"} // deprecated
Number   NestTStat_away_temperature_low_f "Away Temperature Low [%.1f °F]"       {nest="<[thermostats(Upstairs).away_temperature_low_f]"} // deprecated
String   NestTStat_structure_id "Structure ID [%s]"                              {nest="<[thermostats(Upstairs).structure_id]"}
Switch   NestTStat_fan_timer_active "Fan Timer Active [%s]"                      {nest="=[thermostats(Upstairs).fan_timer_active]"}
DateTime NestTStat_fan_timer_timeout "Fan Timer Timeout [%1$tm/%1$td %1$tH:%1$tM]" <calendar> {nest="<[thermostats(Upstairs).fan_timer_timeout]"}
String   NestTStat_name_long "Name Long [%s]"                                    {nest="<[thermostats(Upstairs).name_long]"}
Switch   NestTStat_is_online "Is Online [%s]"                                    {nest="<[thermostats(Upstairs).is_online]"}
DateTime NestTStat_last_connection "Last Connection [%1$tm/%1$td %1$tH:%1$tM]" <calendar> {nest="<[thermostats(Upstairs).last_connection]"}
String   NestTStat_hvac_state "HVAC State [%s]"                                  {nest="<[thermostats(Upstairs).hvac_state]"} // as of 1.7.1
String   NestTStat_where_id "Where ID [%s]"                                      {nest="<[thermostats(Upstairs).where_id]"} // as of 1.9
Switch   NestTStat_is_locked "Is Locked [%s]"                                    {nest="<[thermostats(Upstairs).is_locked]"} // as of 1.9
String   NestTStat_locked_temp_min_f "Locked Temp Min [%s °F]"                   {nest="<[thermostats(Upstairs).locked_temp_min_f]"} // as of 1.9, yes it's a String
String   NestTStat_locked_temp_max_f "Locked Temp Max [%s °F]"                   {nest="<[thermostats(Upstairs).locked_temp_max_f]"} // as of 1.9, yes it's a String
String   NestTStat_locked_temp_min_c "Locked Temp Min [%s °C]"                   {nest="<[thermostats(Upstairs).locked_temp_min_c]"} // as of 1.9, yes it's a String
String   NestTStat_locked_temp_max_c "Locked Temp Max [%s °C]"                   {nest="<[thermostats(Upstairs).locked_temp_max_c]"} // as of 1.9, yes it's a String
String   NestTStat_label "Label [%s]"                                            {nest="=[thermostats(Upstairs).label]"} // as of 1.9
Switch   NestTStat_sunlight_correction_enabled "Sunlight Correction Enabled"     {nest="<[thermostats(Upstairs).sunlight_correction_enabled]"} // as of 1.9
Switch   NestTStat_sunlight_correction_active "Sunlight Correction Active"       {nest="<[thermostats(Upstairs).sunlight_correction_active]"} // as of 1.9
String   NestTStat_where_name "Where Name [%s]"                                  {nest="<[thermostats(Upstairs).where_name]"} // as of 1.9
Number   NestTStat_fan_timer_duration "Fan Timer Duration [%d]"                  {nest="=[thermostats(Upstairs).fan_timer_duration]"} // as of 1.9
String   NestTStat_time_to_target "Time to target temp. [%s min]"                {nest="<[thermostats(Upstairs).time_to_target]"} // as of 1.9
String   NestTStat_time_to_target_training "Time to target training [%s]"        {nest="<[thermostats(Upstairs).time_to_target_training]"} // as of 1.9
String   NestTStat_previous_hvac_mode "Previous HVAC Mode [%s]"                  {nest="<[thermostats(Upstairs).previous_hvac_mode]"} // as of 1.9

/* Smoke+CO detectors - change Master Bedroom to your Smoke+CO detector's name */

String   NestSmoke_name "Name [%s]"                                    {nest="<[smoke_co_alarms(Master Bedroom).name]"}
String   NestSmoke_locale "Locale [%s]"                                {nest="<[smoke_co_alarms(Master Bedroom).locale]"}
String   NestSmoke_structure_id "Structure Id [%s]"                    {nest="<[smoke_co_alarms(Master Bedroom).structure_id]"}
String   NestSmoke_software_version "Software Version [%s]"            {nest="<[smoke_co_alarms(Master Bedroom).software_version]"}
String   NestSmoke_device_id "Device Id [%s]"                          {nest="<[smoke_co_alarms(Master Bedroom).device_id]"}
String   NestSmoke_name_long "Name Long [%s]"                          {nest="<[smoke_co_alarms(Master Bedroom).name_long]"}
Switch   NestSmoke_is_online "Is Online [%s]"                          {nest="<[smoke_co_alarms(Master Bedroom).is_online]"}
String   NestSmoke_where_id "Where ID [%s]"                            {nest="<[smoke_co_alarms(Master Bedroom).where_id]"} // as of 1.9
String   NestSmoke_where_name "Where Name [%s]"                        {nest="<[smoke_co_alarms(Master Bedroom).where_name]"} // as of 1.9
DateTime NestSmoke_last_connection "Last Connection [%1$tm/%1$td/%1$tY %1$tH:%1$tM:%1$tS]" <calendar> {nest="<[smoke_co_alarms(Master Bedroom).last_connection]"}
String   NestSmoke_battery_health "Battery Health [%s]"                {nest="<[smoke_co_alarms(Master Bedroom).battery_health]"}
String   NestSmoke_smoke_alarm_state "Smoke Alarm State [%s]"          {nest="<[smoke_co_alarms(Master Bedroom).smoke_alarm_state]"}
String   NestSmoke_co_alarm_state "CO Alarm State [%s]"                {nest="<[smoke_co_alarms(Master Bedroom).co_alarm_state]"}
String   NestSmoke_ui_color_state "UI Color State [%s]"                {nest="<[smoke_co_alarms(Master Bedroom).ui_color_state]"}
Switch   NestSmoker_is_manual_test_active "Is Manual Test Active [%s]" {nest="<[smoke_co_alarms(Master Bedroom).is_manual_test_active]"}
DateTime NestSmoke_last_manual_test_time "Last Manual Test Time [%1$tm/%1$td/%1$tY %1$tH:%1$tM:%1$tS]" <calendar> {nest="<[smoke_co_alarms(Master Bedroom).last_manual_test_time]"}

/* Nest Cams (available as of openHAB 1.8) -- changing Dining Room to your Cam's name */

String NestCam_device_id "Cam Device ID [%s]"                               {nest="<[cameras(Dining Room).device_id]"}
String NestCam_software_Version "Cam S/W Version [%s]"                      {nest="<[cameras(Dining Room).software_version]"}
String NestCam_where_id "Where ID [%s]"                                     {nest="<[cameras(Dining Room).where_id]"} // as of 1.9
String NestCam_where_name "Where Name [%s]"                                 {nest="<[cameras(Dining Room).where_name]"} // as of 1.9

String NestCam_name "Cam Name [%s]"                                         {nest="<[cameras(Dining Room).name]"}
String NestCam_name_long "Cam Name Long [%s]"                               {nest="<[cameras(Dining Room).name_long]"}
Switch NestCam_is_online "Cam Online [%s]"                                  {nest="<[cameras(Dining Room).is_online]"}
Switch NestCam_is_streaming "Cam Streaming [%s]"                            {nest="=[cameras(Dining Room).is_streaming]"}
Switch NestCam_is_audio_input_enabled "Cam Audio Input Enabled [%s]"        {nest="<[cameras(Dining Room).is_audio_input_enabled]"}
DateTime NestCam_last_is_online_change "Cam Last Online Change [%1$tm/%1$td %1$tH:%1$tM]" {nest="<[cameras(Dining Room).last_is_online_change]"}
Switch NestCam_is_video_history_enabled "Cam Video History Enabled [%s]"    {nest="<[cameras(Dining Room).is_video_history_enabled]"}
String NestCam_web_url "Cam Web URL [%s]"                                   {nest="<[cameras(Dining Room).web_url]"}
String NestCam_app_url "Cam App URL [%s]"                                   {nest="<[cameras(Dining Room).app_url]"}
Switch NestCam_is_public_share_enabled "Cam Public Share Enabled [%s]"      {nest="<[cameras(Dining Room).is_public_share_enabled]"} // as of 1.9
String NestCam_public_share_url "Cam Public Share Url [%s]"                 {nest="<[cameras(Dining Room).public_share_url]"} // as of 1.9
String NestCam_snapshot_url "Cam Snapshot URL [%s]"                         {nest="<[cameras(Dining Room).snapshot_url]"} // as of 1.9
/* All last_event.* binding config strings require a Nest Aware with Video History subscription */
Switch NestCam_last_event_has_sound "Cam Last Event Has Sound [%s]"         {nest="<[cameras(Dining Room).last_event.has_sound]"}
Switch NestCam_last_event_has_motion "Cam Last Event Has Motion [%s]"       {nest="<[cameras(Dining Room).last_event.has_motion]"}
Switch NestCam_last_event_has_person "Cam Last Event Has Person [%s]"       {nest="<[cameras(Dining Room).last_event.has_person]"}
DateTime NestCam_last_event_start_time "Cam Last Event Start Time [%1$tm/%1$td %1$tH:%1$tM]" {nest="<[cameras(Dining Room).last_event.start_time]"}
DateTime NestCam_last_event_end_time "Cam Last Event End Time [%1$tm/%1$td %1$tH:%1$tM]" {nest="<[cameras(Dining Room).last_event.end_time]"}
DateTime NestCam_last_event_urls_expire_time "Cam Last Event URLs Expire Time [%1$tm/%1$td %1$tH:%1$tM]" {nest="<[cameras(Dining Room).last_event.urls_expire_time]"}
String NestCam_last_event_web_url "Cam Last Event Web URL [%s]"             {nest="<[cameras(Dining Room).last_event.web_url]"}
String NestCam_last_event_app_url "Cam Last Event App URL [%s]"             {nest="<[cameras(Dining Room).last_event.app_url]"}
String NestCam_last_event_image_url "Cam Last Event Image Url [%s]"         {nest="<[cameras(Dining Room).last_event.image_url]"}
String NestCam_last_event_animated_image_url "Cam Last Event Animated Image URL [%s]" {nest="<[cameras(Dining Room).last_event.animated_image_url]"}

/* You can reference a device in a specific structure in the case that there are duplicate names 
 * in multiple structures. If you have duplicate-named thermostats or smoke+CO detectors in the
 * same structure, or duplicate-named structures, you will have to rename them at nest.com.
 */

Number NestHome_temp "Home temperature [%.1f °F]"   {nest="<[structures(Home).thermostats(Dining Room).ambient_temperature_f]"}
Number NestCondo_temp "Condo temperature [%.1f °F]" {nest="<[structures(Condo).thermostats(Dining Room).ambient_temperature_f]"}
```

## Binding Examples

* [[Nest Binding Example (new)|Nest-Binding-Example]]

  ![sample](http://watou.github.io/images/nest-binding-example.jpg)

## Logging (openHAB 1.x)

In order to configure logging for this binding to be generated in a separate file add the following to your /configuration/logback.xml file;
```xml
<appender name="NESTFILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
   <file>logs/nest.log</file>
   <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- weekly rollover and archiving -->
      <fileNamePattern>logs/nest-%d{yyyy-ww}.log.zip</fileNamePattern>
      <!-- keep 30 days' worth of history -->
      <maxHistory>30</maxHistory>
   </rollingPolicy>
   <encoder>
     <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %logger{30}[:%line]- %msg%n%ex{5}</pattern>
   </encoder>
</appender>
    
<!-- Choose level ERROR, WARN, INFO, DEBUG or TRACE for detailed logging -->
<logger name="org.openhab.binding.nest" level="TRACE" additivity="false">
   <appender-ref ref="NESTFILE" />
</logger>
```

## Known Issues

1. The binding initiates outbound TCP connections to the Nest infrastructure on port 9553 (however, which outbound ports are used is determined dynamically by the Nest cloud service and may be different from 9553). If the log shows "Connection timed out" or "Exception reading from Nest: Could not get data model", ensure that outbound connections are not being blocked by a firewall.
2. Multiple instance support (allowing the binding to access multiple Nest accounts at once) conflicts with Prohibition 3 of the [Nest Developer Terms of Service](https://developer.nest.com/documentation/cloud/tos), and so is not implemented.
3. The Nest API rounds humidity to 5%, degrees Fahrenheit to whole degrees, and degrees Celsius to 0.5 degrees, so your Nest app will likely show slightly different values from what is available from the API.
4. There is currently a bug where attempting to update an Item with a binding configuration of this form will not work:
```
Number NestCondo_temp "Condo Temperature [%.1f °F]" {nest="=[structures(Condo).thermostats(Dining Room).target_temperature_f]"}
```
While this form should work:
```
Number NestCondo_temp "Condo Temperature [%.1f °F]" {nest="=[thermostats(Dining Room).target_temperature_f]"}
```

## Change Log
### openHAB 1.7.1

* Added the property `hvac_state` that was [added to the Nest API in May 2015](https://developer.nest.com/documentation/cloud/release-notes).  Please note that if you created your Nest client before the addition of this property to the API, your client's permissions may be set to "Thermostat read/write v2," which does not have access to this new property.  To access it, you will have to edit your [client](https://developer.nest.com/clients) to update the permission to v3.  Click the little gear icon to edit your client, click the Change Permissions button, and generate a new `nest:pin_code` for your nest.com account to put into `openhab.cfg` to replace the older `nest:pin_code`. ([#2659](https://github.com/openhab/openhab/pull/2659))
* Very rarely, some updates to DateTime items would attempt to echo back as changes to the Nest API, generating log errors. ([#2930](https://github.com/openhab/openhab/pull/2930))

### openHAB 1.8.0

* Quiesce logging on status poll ([#3111](https://github.com/openhab/openhab/pull/3111))
* Added support for monitoring and turning on or off streaming from your Nest Cams. ([#3232](https://github.com/openhab/openhab/pull/3232))
* Added optional `nest:timeout` config parameter ([#3692](https://github.com/openhab/openhab/pull/3692))

### openHAB 1.8.1

* Allow setting setpoints with decimals ([#3950](https://github.com/openhab/openhab/pull/3950))

### openHAB 1.x addons 1.9.0

* May 2016 API updates ([#4383](https://github.com/openhab/openhab/pull/4383))
* Silently skip referenced devices that weren't provided, presumably owing to incomplete Product permissions set at nest.com ([#4649](https://github.com/openhab/openhab/pull/4649))
* October 2016 API updates ([#4716](https://github.com/openhab/openhab/pull/4716))
* November 2016 API updates ([#4873](https://github.com/openhab/openhab/pull/4873))
