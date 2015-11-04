Documentation of the RFXCOM binding Bundle

## Introduction

Binding should be compatible at least with RFXtrx433 USB 433.92MHz transceiver, which contains both receiver and transmitter functions. 

Supports RF 433 Mhz protocols like: HomeEasy, Cresta, X10, La Crosse, OWL, CoCo ([KlikAanKlikUit](http://www.klikaanklikuit.nl)), PT2262, Oregon e.o. <br>
See further information from http://www.rfxcom.com

RFXCOM binding support currently Blinds1, Control, Current, Curtain1, Energy, Humidity, Interface, Lighting1, Lighting2, Lighting4, Lighting5, Lighting6, Factory, Interface, Rain, Rfy, Security1, TemperatureHumidity, Temperature, Thermostat1, Transmitter, Wind packet types. 

For installation of the binding, please see Wiki page [[Bindings]].

## Binding Configuration

First of all you need to configure the following values in the openhab.cfg file (in the folder '${openhab_home}/configurations').

    ############################### RFXCOM Binding #########################################
    
    # Serial port of RFXCOM interface
    # Valid values are e.g. COM1 for Windows and /dev/ttyS0 or /dev/ttyUSB0 for Linux
    rfxcom:serialPort=
    
    # Set mode command for controller (optional)
    # E.g. rfxcom:setMode=0D000000035300000C2F00000000 
    rfxcom:setMode=

The `rfxcom:serialPort` value is the identification of the serial port on the host system where RFXCOM controller is connected, e.g. 
"COM1" on Windows,"/dev/ttyS0" on Linux or "/dev/tty.PL2303-0000103D" on Mac.

NOTE: On Linux, should the RFXCOM device be added to the `dialout` group, you may get an error stating the the serial port cannot be opened when the RfxCom plugin tries to load.  You can get around this by adding the `openhab` user to the `dialout` group like this: `usermod -a -G dialout openhab`.

The `rfxcom:setMode` value is optional. Set mode command can be used to configure RFXCOM controller to listening to various receiver protocols. This is very useful because the receiver will become more sensitive when only the required protocols are enabled. You can use the RFXmngr application to get the valid configuration command. Command must be a 28 characters (14 bytes) hexadecimal string.  You can also use this to get the Device Ids needed below.

## Finding the correct values

If you want to add cheap 433 MHz devices like PT2622 remotes, contacts, sensors or wireless outlets and don't know the correct values, you can start OpenHAB in debug mode, press the buttons on the original remote or act on the sensor.

To pair the entry-level wireless remote outlets (Elro, Intertechno, Intertek, Pollin, ...) you can do the following:

- Enable Lighting4 protocol using the RFXMgr or `setMode`
- Run OpenHAB in debug mode
- Press a button on the original remote controller (here I press `A ON`) and read the DeviceId `Id` and the pulse width `Pulse` from the output:

`21:35:20.096 [DEBUG] [.b.r.internal.RFXComConnection:148  ] - Data received:`
`- Packet type = LIGHTING4`
`- Id = 1285`
`- Command = ON`
`- Pulse = 318`

- You can use these values to configure your binding (see below) to receive or send with this data.

## Item Binding Configuration

In order to bind an item to RFXCOM device, you need to provide configuration settings. The easiest way to do so is to add some binding information in your item file (in the folder configurations/items). The syntax of the binding configuration strings accepted is the following:

    in:  rfxcom="<DeviceId:ValueSelector"
    out: rfxcom=">DeviceId:PacketType.SubType:ValueSelector"

where `DeviceID` is a valid wireless device identifier in decimal (**not hexadecimal !**).

- Lighting1 format: `SensorId.UnitCode`
    e.g. B.1, B.2 or B.0 for group functions

- Lighting2 formats: `SensorId.UnitCode`
    e.g. 636602.1 or 636602.0 for group functions

- Lighting4 formats: `SensorId`
    e.g. 1285 for a PT2622 remote (House 11111, Device D)

- Lighting5 format: `SensorId.UnitCode`
    e.g. 636602.1

- Lighting6 format: `SensorId.GroupCode.UnitCode `
    e.g. 257.B.1, 64343.B.2 or 636602.H.5 

- Curtain1 format: `SensorId.UnitCode`
    e.g. P.1 see RFXCOM documentation

- TemperatureHumidity, Current, Energy,... format: `SensorId`
    e.g. 2561

Examples, how to configure your items:

    Weather Station Example
    Number OutdoorTemperature { rfxcom="<2561:Temperature" }
    Number OutdoorHumidity { rfxcom="<2561:Humidity" }
    Number RainRate	{ rfxcom="<30464:RainRate" }
    Number WindSpeed	{ rfxcom="<19968:WindSpeed" }

    Switch Btn1 { rfxcom="<636602.1:Command" }
    Number Btn1SignalLevel { rfxcom="<636602.1:SignalLevel" }
    Dimmer Btn1DimLevel { rfxcom="<636602.1:DimmingLevel" }
    String Btn2RawData { rfxcom="<636602.2:RawData" }
    Switch ChristmasTreeLights { rfxcom">636602.1:LIGHTING2.AC:Command" }
    Rollershutter CurtainDownstairs { rfxcom=">P.1:CURTAIN1.HARRISON:Shutter" }
    Rollershutter ShutterBedroom { rfxcom=">1.0.0.1:RFY.RFY:Shutter" }
    
    SECURITY1.X10_SECURITY_MOTION  example
    Switch swMotion { rfxcom="<4541155:Motion" }
    Number MSensor_Bat { rfxcom="<4541155:BatteryLevel" }

    LIGHTING4.PT2262  example
    Switch swWallController { rfxcom="<1285:Command" } // receive wireless wall switch
    Switch pCoffeeMachine { rfxcom=">1285.315:LIGHTING4.PT2262:Command" } // control wireless outlet

    THERMOSTAT1  example
    Number RFXTemp_Living { rfxcom=<30515:Temperature" 
    Number RFXTemp_LivingSP { rfxcom="<30515:SetPoint" }
    Contact RFXTemp_LivingRoom_Stat { rfxcom="<30515:Contact" }     	

    LIGHTWAVERF example
    Switch Light1 { rfxcom=">3155730.3:LIGHTING5.LIGHTWAVERF:Command"}
    Dimmer Light2 "Light2 [%d %%]" { rfxcom=">3155730.4:LIGHTING5.LIGHTWAVERF:DimmingLevel"  }

    LIGHTWAVERF Mood Switch example
    Number Button_MoodSwitch { rfxcom="<15942554.16:Mood" }

    OWL CM160 Energy Monitor example
    Number Owl_InstantAmps { rfxcom="<63689:InstantAmps"}
    Number Owl_TotalAmpHours { rfxcom="<63689:TotalAmpHours"  }

    OWL CM113 Energy Monitor example
    Number Owl_Amps { rfxcom="<35072:Channel2Amps" }


`PacketType.SubType` specify packet and sub type information ...

<table>
  <tr><td><b>PacketType.SubType</b></td><td><b>Description</b></td><td><b>ValueSelector</b></td></tr>
  <tr><td>LIGHTING1.X10</td><td>working</td><td>Command</td></tr>
  <tr><td>LIGHTING1.ARC</td><td>working</td><td>Command</td></tr>
  <tr><td>LIGHTING1.AB400D</td><td>Untested</td><td></td></tr>
  <tr><td>LIGHTING1.WAVEMAN</td><td>Untested</td><td></td></tr>
  <tr><td>LIGHTING1.EMW200</td><td>Untested</td><td></td></tr>
  <tr><td>LIGHTING1.IMPULS</td><td>working</td><td>Command</td></tr>
  <tr><td>LIGHTING1.RISINGSUN</td><td>Untested</td><td></td></tr>
  <tr><td>LIGHTING1.PHILIPS</td><td>Untested</td><td></td></tr>
  <tr><td>LIGHTING1.ENERGENIE</td><td>working</td><td>Command</td></tr>
  <tr><td>LIGHTING2.AC</td><td>working</td><td>Command, DimmingLevel</td></tr>
  <tr><td>LIGHTING2.HOME_EASY_EU</td><td>working</td><td>Command,DimmingLevel</td></tr>
  <tr><td>LIGHTING2.ANSLUT</td><td>Untested</td><td></td></tr>
  <tr><td>LIGHTING4.PT2262</td><td>working</td><td>Command</td></tr>
  <tr><td>LIGHTING5.LIGHTWAVERF</td><td>working</td><td>Command, DimmingLevel</td></tr>
  <tr><td>LIGHTING6.BLYSS</td><td>working</td><td>Command</td></tr>
  <tr><td>CURTAIN1.HARRISON</td><td>Harrison curtain rail, e.g. Neta 12</td><td>Shutter</td></tr>
   <tr><td>TEMPERATURE.La Crosse TX17</td><td>working</td><td></td></tr>
  <tr><td>TEMPERATUREHUMIDITY.Oregon 2.1<br>THGN122_123_132_THGR122_228_238_268</td><td>working</td><td></td></tr>
  <tr><td>TEMPERATUREHUMIDITY.THGN800_THGR810</td><td>working</td><td></td></tr>
  <tr><td>TEMPERATUREHUMIDITY.RTGR328</td><td>Untested</td><td></td></tr>
  <tr><td>TEMPERATUREHUMIDITY.THGR328</td><td>Untested</td><td></td></tr>
  <tr><td>TEMPERATUREHUMIDITY.WTGR800</td><td>Untested</td><td></td></tr>
  <tr><td>TEMPERATUREHUMIDITY.<br>THGR918_THGRN228_THGN50</td><td>Untested</td><td></td></tr>
  <tr><td>TEMPERATUREHUMIDITY.TFA_TS34C__CRESTA</td><td>Working</td><td>Temperature, Humidity, HumidityStatus, BatteryLevel, SignalLevel</td></tr>
  <tr><td>TEMPERATUREHUMIDITY.<br>WT260_WT260H_WT440H_WT450_WT450H</td><td>Untested</td><td></td></tr>
  <tr><td>TEMPERATUREHUMIDITY.VIKING_02035_02038</td><td>Untested</td><td></td></tr>
  <tr><td>SECURITY1.X10_SECURITY_MOTION</td><td>working</td><td>Motion</td></tr>
  <tr><td>THERMOSTAT1</td><td>Digimax 210 working</td><td>Temperature, SetPoint, Contact</td></tr>

  <tr><td>ENERGY.ELEC1</td><td>Owl CM113 Working</td><td>Channel1Amps, Channel2Amps, Channel3Amps</td></tr>
  <tr><td>ENERGY.ELEC2</td><td>Owl CM160 Working</td><td>InstantAmps, TotalAmpHours</td></tr>
  <tr><td>RFY.RFY</td><td>Working</td><td></td>

</table>

`ValueSelector` specify ...

<table>
  <tr><td><b>Value selector</b></td><td><b>Valid OpenHAB data type</b></td><td><b>Information</b></td></tr>
  <tr><td>RawData</td><td>StringItem</td><td></td></tr>
  <tr><td>Command</td><td>SwitchItem</td><td>ON, OFF, GROUP_ON, GROUP_OFF</td></tr>
  <tr><td>SignalLevel</td><td>NumberItem</td><td></td></tr>
  <tr><td>DimmingLevel</td><td>DimmerItem</td><td>UP, DOWN, PERCENTAGE</td></tr>
  <tr><td>Temperature</td><td>NumberItem</td><td></td></tr>
  <tr><td>Humidity</td><td>NumberItem</td><td></td></tr>
  <tr><td>HumidityStatus</td><td>StringItem</td><td></td></tr>
  <tr><td>BatteryLevel</td><td>NumberItem</td><td></td></tr>
  <tr><td>Shutter</td><td>RollershutterItem</td><td>OPEN, CLOSE, STOP</td></tr>
  <tr><td>Motion</td><td>SwitchItem</td><td>MOTION, NO_MOTION</td></tr>
  <tr><td>Voltage</td><td>NumberItem</td><td></td></tr>
  <tr><td>SetPoint</td><td>NumberItem</td><td></td></tr>
  <tr><td>Pressure</td><td>NumberItem</td><td></td></tr>
  <tr><td>Forecast</td><td>NumberItem</td><td></td></tr>
  <tr><td>RainRate</td><td>NumberItem</td><td></td></tr>
  <tr><td>RainTotal</td><td>NumberItem</td><td></td></tr>
  <tr><td>WindDirection</td><td>NumberItem</td><td></td></tr>
  <tr><td>WindSpeed</td><td>NumberItem</td><td></td></tr>
  <tr><td>Gust</td><td>NumberItem</td><td></td></tr>
  <tr><td>ChillFactor</td><td>NumberItem</td><td></td></tr>
  <tr><td>InstantPower</td><td>NumberItem</td><td></td></tr>
  <tr><td>TotalUsage</td><td>NumberItem</td><td></td></tr>
  <tr><td>Voltage</td><td>NumberItem</td><td></td></tr>
  <tr><td>InstantAmps</td><td>NumberItem</td><td></td></tr>
  <tr><td>TotalAmpHours</td><td>NumberItem</td><td></td></tr>
  <tr><td>Channel1Amps</td><td>NumberItem</td><td></td></tr>
  <tr><td>Channel2Amps</td><td>NumberItem</td><td></td></tr>
  <tr><td>Channel3Amps</td><td>NumberItem</td><td></td></tr>
  <tr><td>Status</td><td>StringItem</td><td></td></tr>
  <tr><td>Mood</td><td>NumberItem</td><td></td></tr>
  <tr><td>Contact</td><td>ContactItem</td><td></td></tr>
</table>