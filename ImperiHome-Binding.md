# ImperiHome Binding

*Looking for MAINTAINER. Revenz is inactive and not reachable*

**Please Note:** 
This binding is in alpha stage; currently available for download via the [OpenHAB-Forum](https://groups.google.com/d/msg/openhab/TWrvCd1fens/mO83ymI772sJ).

 * [[Purpose|ImperiHome-Binding#purpose]]
 * [[Installation|ImperiHome-Binding#installation]]
 * [[Configuration|ImperiHome-Binding#configuration]]
  * [[Items|ImperiHome-Binding#items]]
    * [[Binding Format|ImperiHome-Binding#binding-format]]
    * [[Examples|ImperiHome-Binding#examples]]

## Purpose

The binding will enable you to use the comercial [ImperiHome](http://www.imperihome.com/) UI.

![](http://www.imperihome.com/wp-content/main-screens.png)

## Installation
Copy the .jar files provided into the ./addons directory of your OH installation.

[[Page Top|ImperiHome-Binding#imperihome-binding]]

## Configuration
The binding provides the API for ImperiHome to load the devices from openHAB and to control it.

### Items
#### Binding Format
```
{imperihab="room:[#room],label:[#label],type:[#type],watts:[#wattsitem],unit:[#unit],persist:[#persist],invert:[#invert]"}
```
<table>
  <tr><td>#room</td><td>the room you want the item to appear under in ImperiHome</td></tr>
  <tr><td>#label</td><td>Optional, the name of the item to appear in ImperiHome, if not specified the name will be the item name replacing _ with " ", so Outside_Porch_Light would become "Outside Porch Light" automatically</td></tr>
  <tr><td>#type</td><td>Optional, only needed if the type of the device cannot be guessed from the item.  It first tries to find the type based on the values it support OpenClose, OnOff, Percentage etc and there's some best guesses for the item names e.g. if item name contains "Humidity" it thinks its a humidity sensor).  If none of these work, or if you want to override the type it guesses, you can specify it.  Using the device types for imperihome: 
<table>
  <tr><th>Device type string</th><th>Description</th></tr>
  <tr><td>DevCamera</td><td>MJPEG IP Camera</td></tr>
  <tr><td>DevCO2</td><td>CO2 sensor</td></tr>
  <tr><td>DevCO2Alert</td><td>CO2 Alert sensor</td></tr>
  <tr><td>DevDimmer</td><td>Dimmable light</td></tr>
  <tr><td>DevDoor</td><td>Door / window security sensor</td></tr>
  <tr><td>DevElectricity</td><td>Electricity consumption sensor</td></tr>
  <tr><td>DevFlood</td><td>Flood security sensor</td></tr>
  <tr><td>DevGenericSensor</td><td>Generic sensor (any value)</td></tr>
  <tr><td>DevHygrometry</td><td>Hygro sensor</td></tr>
  <tr><td>DevLock</td><td>Door lock</td></tr>
  <tr><td>DevLuminosity</td><td>Luminance sensor</td></tr>
  <tr><td>DevMotion</td><td>Motion security sensor</td></tr>
  <tr><td>DevMultiSwitch</td><td>Multiple choice actuator</td></tr>
  <tr><td>DevNoise</td><td>Noise sensor</td></tr>
  <tr><td>DevPressure</td><td>Pressure sensor</td></tr>
  <tr><td>DevRain</td><td>Rain sensor</td></tr>
  <tr><td>DevScene</td><td>Scene (launchable)</td></tr>
  <tr><td>DevShutter</td><td>Shutter actuator</td></tr>
  <tr><td>DevSmoke</td><td>Smoke security sensor</td></tr>
  <tr><td>DevSwitch</td><td>Standard on/off switch</td></tr>
  <tr><td>DevTemperature</td><td>Temperature sensor</td></tr>
  <tr><td>DevThermostat</td><td>Thermostat</td></tr>
  <tr><td>DevUV</td><td>UV sensor</td></tr>
  <tr><td>DevWind</td><td>Wind sensor</td></tr>
</table>
</td></tr>
  <tr><td>#wattsitem</td><td>Optional, this lets you specify another item to be the "energy" value for an item, e.g. for a z-wave power outlet Switch Item, you can link this to the Number item that has the power reading.  Then in imperihome it will show the power usage for that switch.</td></tr>
  <tr><td>#persist</td><td>Optional, this lets you specify another persist store to use instead of the default configured in openHAB.</td></tr>
  <tr><td>#unit</td><td>Optional lets you specify the units for a numeric value</td></tr>
  <tr><td>#invert</td><td>Optional (true|false), this lets you invert an on/off, open/closed state of an item.  This is helpful if ImperiHome is showing the wrong state for a door.  E.g. a garage door appears open when closed etc. </td></tr>
</table>

[[Page Top|ImperiHome-Binding#imperihome-binding]]

#### Examples
```
Number zWaveSensor23_1 "L1 [%.1f W]" <energy> (gZWaveNode23, gPower) {zwave="23:1:command=METER", imperihab="room:Keller,label:Verbrauch L1,type:DevElectricity,watts:zWaveSensor23_1"}
Dimmer zWaveLightOGBedroom "Licht [%d %%]" <light> (gZWaveNode20, gLights, gHomeOGBedroom) {zwave="20:0:command=SWITCH_MULTILEVEL", imperihab="room:Schlafzimmer,label:Licht,type:DevDimmer,watts:zWaveLightOGBedroom"}
Rollershutter zWaveShutterEGLivingroomLeft "Rollladen" <rollershutter> (gZWaveNode16, gHomeShuttersEG, gHomeEGLivingRoom) {zwave="16:0:command=SWITCH_MULTILEVEL", imperihab="room:Wohnraum,label:Rollladen,type:DevShutter,watts:zWaveShutterEGLivingroomLeft"}
Number zWaveSensor2Temperatur "Temperatur EG [%.1f °C]" <temperature> (gZWaveNode2, gTemperature, gHomeEGTV) {zwave="2:3:command=SENSOR_MULTILEVEL, sensor_type=1, sensor_scale=0", imperihab="room:TV,label:Temperatur,type:DevTemperature,watts:zWaveSensor2Temperatur"} 
```
[[Page Top|ImperiHome-Binding#imperihome-binding]]