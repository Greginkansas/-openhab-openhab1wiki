### Introduction

The Astro binding is used for calculating many DateTime and positional values for sun and moon and for scheduling of events.

### Installation

`sudo apt-get update`
`sudo apt-get install openhab-addon-binding-astro`

### Configuration in openhab.cfg
```
############################## Astro Binding ##############################
#
# Your latitude
astro:latitude=nn.nnnnnn
 
# Your longitude
astro:longitude=nn.nnnnnn
 
# Refresh interval for azimuth and elevation calculation in seconds
# (optional, defaults to disabled)
astro:interval=nnn
```

### Available Items
**Important:** type and property are case sensitive! So enter the values exactly as shown.

* **planet** `sun`
    * **type** `rise, set, noon, night, morningNight, astroDawn, nauticDawn, civilDawn, astroDusk, nauticDusk, civilDusk, eveningNight, daylight` ([Order of these types](https://github.com/openhab/openhab/wiki/Astro-Binding---Schedules))
        * **property** `start, end` (DateTime), `duration` (Number)
    * **type** `position`
        * **property** `azimuth, elevation` (Number)
    * **type** `zodiac`
        * **property** `start, end` (DateTime), `sign` (String)
    * **type** `season`
        * **property**: `spring, summer, autumn, winter` (DateTime), `name` (String)
    * **type** `eclipse`
        * **property**: `total, partial, ring` (DateTime)
* **planet** `moon`
    * **type** `rise, set`
        * **property** `start, end` (DateTime), `duration` (Number), **Note:** start and end is always equal, duration always 0.
    * **type** `phase`
        * **property**: `firstQuarter, thirdQuarter, full, new` (DateTime), `age, illumination` (Number), `name` (String)
    * **type** `eclipse`
        * **property**: `total, partial` (DateTime)
    * **type** `distance`
        * **property**: `date` (DateTime), `kilometer, miles` (Number)
    * **type** `perigee`
        * **property**: `date` (DateTime), `kilometer, miles` (Number)
    * **type** `apogee`
        * **property**: `date` (DateTime), `kilometer, miles` (Number)
    * **type** `zodiac`
        * **property** `sign` (String)
    * **type** `position`
        * **property** `azimuth, elevation` (Number)

**offset** (optional, taken into account for every DateTime property)  
offset in minutes to the calculated time

You can bind a property to different item types, which has a special meaning in the binding. If you bind a DateTime property (start, end, ...) to a DateTime Item, the DateTime is simply displayed. If you bind it to a Switch, a event is scheduled and the state of the Switch is updated to ''ON'', immediately followed by a ''OFF'' at the calculated time. You can even specify a offset for the event and bind multiple items to the same property.

The position items (azimuth, elevation) and moon items (phase, distance, perigee, apogee, zodiac) are updated at the configured refresh interval in openhab.cfg.

###Sun examples
```
// shows the sunrise
DateTime Sunrise_Time  "Sunrise [%1$tH:%1$tM]"  {astro="planet=sun, type=rise, property=start"}

// schedules a event which starts at sunrise, updating the Switch with ON, followed by a OFF
Switch Sunrise_Event   {astro="planet=sun, type=rise, property=start"}

// schedules a event which starts 10 minutes AFTER sunrise
Switch Sunrise_Event   {astro="planet=sun, type=rise, property=start, offset=10"}

// schedules a event which starts 10 minutes BEFORE sunrise
Switch Sunrise_Event   {astro="planet=sun, type=rise, property=start, offset=-10"}

// shows the sunset
DateTime Sunset_Time   "Sunset [%1$tH:%1$tM]"   {astro="planet=sun, type=set, property=end"}

// schedules a event which starts 30 minutes BEFORE sunset:
Switch Sunset_Event    {astro="planet=sun, type=set, property=end, offset=-30"}

// displays the start, end and duration of the astronomical dawn
DateTime Astronomical_Dawn_Start        "Astronomical Dawn Start [%1$tH:%1$tM]"  {astro="planet=sun, type=astroDawn, property=start"}
DateTime Astronomical_Dawn_End          "Astronomical Dawn End [%1$tH:%1$tM]"    {astro="planet=sun, type=astroDawn, property=end"}
// duration in minutes
Number   Astronomical_Dawn_Duration     "Astronomical Dawn Duration [%f]"        {astro="planet=sun, type=astroDawn, property=duration"}
// duration formatted to a string, e.g. 02:32 (2 hours, 32 minutes)
String   Astronomical_Dawn_Duration_Str "Astronomical Dawn Duration [%s]"        {astro="planet=sun, type=astroDawn, property=duration"}


// azimuth and elevation
Number   Azimuth        "Azimuth [%.2f]"     {astro="planet=sun, type=position, property=azimuth"}
Number   Elevation      "Elevation [%.2f]"   {astro="planet=sun, type=position, property=elevation"}


// zodiac
DateTime Zodiac_Start   "Zodiac Start [%1$td.%1$tm.%1$tY]"   {astro="planet=sun, type=zodiac, property=start"}
DateTime Zodiac_End     "Zodiac End [%1$td.%1$tm.%1$tY]"     {astro="planet=sun, type=zodiac, property=end"}
String   Zodiac_Sign    "Current zodiac [%s]"                {astro="planet=sun, type=zodiac, property=sign"}


// season
String Season_Name      "Season [%s]"                             {astro="planet=sun, type=season, property=name"}
DateTime Season_Spring  "Spring [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"  {astro="planet=sun, type=season, property=spring"}
DateTime Season_Summer  "Summer [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"  {astro="planet=sun, type=season, property=summer"}
DateTime Season_Autumn  "Autumn [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"  {astro="planet=sun, type=season, property=autumn"}
DateTime Season_Winter  "Winter [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"  {astro="planet=sun, type=season, property=winter"}


// eclipse
DateTime Sun_Eclipse_Total   "Sun total eclipse [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"   {astro="planet=sun, type=eclipse, property=total"}
DateTime Sun_Eclipse_Partial "Sun partial eclipse [%1$td.%1$tm.%1$tY %1$tH:%1$tM]" {astro="planet=sun, type=eclipse, property=partial"}
DateTime Sun_Eclipse_Ring    "Sun ring eclipse [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"    {astro="planet=sun, type=eclipse, property=ring"}
```

###Moon examples  
```
// rise, set
DateTime Moonrise_Time   "Moonrise [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"  {astro="planet=moon, type=rise, property=start"}
DateTime Moonset_Time    "Moonset [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"   {astro="planet=moon, type=set, property=end"}


// phase
DateTime  Moon_First_Quarter "First Quarter [%1$td.%1$tm.%1$tY %1$tH:%1$tM]" {astro="planet=moon, type=phase, property=firstQuarter"}
DateTime  Moon_Third_Quarter "Third Quarter [%1$td.%1$tm.%1$tY %1$tH:%1$tM]" {astro="planet=moon, type=phase, property=thirdQuarter"}
DateTime  Moon_Full          "Full moon [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"     {astro="planet=moon, type=phase, property=full"}
DateTime  Moon_New           "New moon [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"      {astro="planet=moon, type=phase, property=new"}
Number    Moon_Age           "Moon Age [%.0f days]"        {astro="planet=moon, type=phase, property=age"}
Number    Moon_Illumination  "Moon Illumination [%.1f %%]" {astro="planet=moon, type=phase, property=illumination"}
String    Moon_Phase_Name    "Moonphase [%s]"              {astro="planet=moon, type=phase, property=name"}


// distance
Number   Moon_Distance_K    "Moon distance [%.2f km]"    {astro="planet=moon, type=distance, property=kilometer"}
Number   Moon_Distance_M    "Moon distance [%.2f miles]" {astro="planet=moon, type=distance, property=miles"}
DateTime Moon_Distance_Time "Moon distance from [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"  {astro="planet=moon, type=distance, property=date"}


// eclipse
DateTime Moon_Eclipse_Total    "Moon total eclipse [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"    {astro="planet=moon, type=eclipse, property=total"}
DateTime Moon_Eclipse_Partial  "Moon partial eclipse [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"  {astro="planet=moon, type=eclipse, property=partial"}


// perigee
Number   Moon_Perigee_K     "Moon perigee [%.2f km]"    {astro="planet=moon, type=perigee, property=kilometer"}
Number   Moon_Perigee_M     "Moon perigee [%.2f miles]" {astro="planet=moon, type=perigee, property=miles"}
DateTime Moon_Perigee_Time  "Moon perigee from [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"    {astro="planet=moon, type=perigee, property=date"}


// apogee
Number   Moon_Apogee_K      "Moon apogee [%.2f  km]"    {astro="planet=moon, type=apogee, property=kilometer"}
Number   Moon_Apogee_M      "Moon apogee [%.2f miles]"  {astro="planet=moon, type=apogee, property=miles"}
DateTime Moon_Apogee_Time   "Moon apogee from [%1$td.%1$tm.%1$tY %1$tH:%1$tM]"     {astro="planet=moon, type=apogee, property=date"}


// moon zodiac
String   Moon_Zodiac_Sign   "Moon zodiac [%s]"          {astro="planet=moon, type=zodiac, property=sign"}


// moon azimuth and elevation
Number   Moon_Azimuth       "Moon azimuth [%.2f]"       {astro="planet=moon, type=position, property=azimuth"}
Number   Moon_Elevation     "Moon elevation [%.2f]"     {astro="planet=moon, type=position, property=elevation"}


// schedules a event at full moon
Switch   Moon_Full_Event    {astro="planet=moon, type=phase, property=full"}

// schedules a event 10 minutes BEFORE new moon
Switch   Moon_New_Event     {astro="planet=moon, type=phase, property=new, offset=-10"}
```

If you like to have the season name, zodiac sign and the moon phase name in your own language, use a map.  
Example for german translation:
```
String Zodiac_Sign_Ger      "Tierkreiszeichen [MAP(zodiac.map):%s]"  {astro="planet=sun, type=zodiac, property=sign"}
String Season_Name_Ger      "Jahreszeit [MAP(season.map):%s]"        {astro="planet=sun, type=season, property=name"}
String Moon_Phase_Ger       "Mondphase [MAP(moon.map):%s]"           {astro="planet=moon, type=phase, property=name"}
String Moon_Zodiac_Sign_Ger "Mondzeichen [MAP(zodiac.map):%s]"       {astro="planet=moon, type=zodiac, property=sign"}
```
**zodiac.map**  
```
Aries=Widder
Taurus=Stier
Gemini=Zwilling
Cancer=Krebs
Leo=Löwe
Virgo=Jungfrau
Libra=Waage
Scorpio=Skorpion
Sagittarius=Schütze
Capricorn=Steinbock
Aquarius=Wassermann
Pisces=Fisch
```
**season.map**  
```
Spring=Frühling
Summer=Sommer
Autumn=Herbst
Winter=Winter
```
**moon.map**
```
New=Neumond
Waxing\u0020Crescent=zunehmender Halbmond
First\u0020Quarter=erstes Viertel
Waxing\u0020Gibbous=zunehmender Mond
Full=Vollmond
Waning\u0020Gibbous=abnehmender Mond
Third\u0020Quarter=letztes Viertel
Waning\u0020Crescent=abnehmender Halbmond
```

### Example Rules
Rule at sunrise:
```Xtend
rule "Example Rule at sunrise"
when
    Item Sunrise_Event received update ON
then
    ...
end
```

Rule to close all RollerShutters after sunset and the outside temperature is lower than 5 degrees:
```Xtend
rule "Close RollerShutters if cold after sunset"
when
    Item Temp_Outside changed
then
    if (now.isAfter((Sunset_Time.state as DateTimeType).calendar.timeInMillis) &&
       (Temp_Outside.state as DecimalType).intValue < 5) {
		
       RollerShutters?.members.forEach(r | sendCommand(r, DOWN))

    }
end
```

Let's say, you know that the sun is shining through your livingroom window between Azimuth 100 and 130. If it's summer you want to close the RollerShutter.
```Xtend
rule "Autoshading RollerShutter"
when
    Item Azimuth changed
then
    var int azimuth = (Azimuth.state as DecimalType).intValue
	
    if (azimuth > 100 && azimuth < 130) {
      sendCommand(Rollershutter_Livingroom, DOWN)
    }

    ...
end
```

### Troubleshooting
I assume, the binding is in your addons folder. It populates the astro items at startup and with scheduled jobs.

* In the openHab logfile there must be a entry like this: `AstroConfig[latitude=xx.xxxx,longitude=xx.xxxx,interval=...,systemTimezone=...,daylightSavings=...]`  
If this entry does not exist, there is a problem in your openhab.cfg.

* If the items are still not populated, switch the binding to DEBUG mode and start openHab. Now you should see for every astro item a entry in your logfile: `Adding item ... with AstroBindingConfig[planet=..., type=..., property=...]`  
If you don't see these entries, check your item file.

* If the maps for translation are not working, there might be a file encoding problem. Download the german example maps and edit the entries.

# Screenshots
![](https://dl.dropboxusercontent.com/u/1781347/wiki/2015-01-04%2016_27_28-openHAB.png)

# Download
[German maps download] (https://drive.google.com/file/d/0Bw7zjCgsXYnHZXNNeU5XY2FTMGc/edit?usp=sharing)