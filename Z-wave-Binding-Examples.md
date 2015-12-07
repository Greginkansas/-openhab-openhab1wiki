**TODO/Work in progress: This page needs sorting alphabetically, into categories and with headings**

Here are some examples of valid binding configuration strings, as defined in the items configuration file:

    /* Some general z-wave statistics */
    
    Number ZwaveStatsSOF "Number Start of Frames[%s]" (gZwaveStats) {zwave="1:command=info,item=sof"}
    Number ZwaveStatsACK "Number of Acknowledgments [%s]" (gZwaveStats) {zwave="1:command=info,item=ack"}
    Number ZwaveStatsCAN "Number of CAN [%s]" (gZwaveStats) {zwave="1:command=info,item=can"}
    Number ZwaveStatsNAK "Number of NAK [%s]" (gZwaveStats) {zwave="1:command=info,item=nak"}
    Number ZwaveStatsOOF "Number of OOF [%s]" (gZwaveStats) {zwave="1:command=info,item=oof"}
    Number ZwaveStatsTimeout "Number of Time-outs [%s]" (gZwaveStats) {zwave="1:command=info,item=time_out"}
    String ZwaveNode01HomeID	"Home ID [%s]" (gZwaveNode01) {zwave="1:command=info,item=home_id"}
    Number ZwaveNode01NetworkID	"Node ID [%s]" (gZwaveNode01) {zwave="1:command=info,item=node_id"}
    
    /* A dimmer and a contact */
    
    Dimmer Light_Corridor_Dimmer "Hallway Dimmer [%d %%]" (GF_Corridor) {zwave="6"}
    Contact Door_Corridor_Switch "Front door sensor [MAP(nl.map):%s]" (GF_Corridor) {zwave="21:command=sensor_binary,respond_to_basic=true"} 
    Number Door_Corridor_Battery "Front door sensor battery level [%d %%]" (GF_Corridor) { zwave="21:command=battery" }
    
    /* A node with multiple endpoints */
    
    Switch Mech_Vent			"Mechanical ventilation middle."	(GF_Kitchen) {zwave="11:1"}
    Switch Mech_Vent_High		"Mechanical ventilation high."	(GF_Kitchen) {zwave="11:2"}
    
    /* A Everspring ST814 temperature and humidity sensor as example of sensor_multilevel */
   
    Number T_AZH  "Temperature [%.1f °C]" (AZH) {zwave="32:1:command=sensor_multilevel,sensor_type=1" }
    Number RH_AZH "Humidity    [%.0f %%]" (AZH) { zwave="32:1:command=sensor_multilevel,sensor_type=5" }
    
    /* A fibaro wall plug with energy meter. */
    
    Switch Coffee_Kitchen_Switch "Coffee machine" (GF_Cellar) {zwave="18:command=switch_binary"} 
    Number Coffee_Kitchen_Power "Coffee machine power consumption [%.1f W]" (GF_Cellar,GF_Energy) { zwave="18:command=sensor_multilevel" }
    Number Coffee_Kitchen_Energy "Coffee machine total energy usage  [%.2f KWh]" (GF_Cellar) { zwave="18:command=meter" }

More Examples:

Power Bar:
    
    /* A six node power bar with integrated energy and power meter and different intervals. */
    
    Switch Switch_Powerbar_Subwoofer "Subwoofer" (GF_Living) {zwave="26:1:command=switch_binary"} 
    Switch Switch_Powerbar_Reiceiver "Receiver" (GF_Living) {zwave="26:2:command=switch_binary"} 
    Switch Switch_Powerbar_DVD "DVD" (GF_Living) {zwave="26:3:command=switch_binary"} 
    Switch Switch_Powerbar_TV "TV" (GF_Living) {zwave="26:4:command=switch_binary"} 
    Switch Switch_Powerbar_Xbox "XBOX-360" (GF_Living) {zwave="26:5:command=switch_binary"} 
    Switch Switch_Powerbar_PC "Mediacenter" (GF_Living) {zwave="26:6:command=switch_binary"} 
    
    Number Power_Powerbar_Subwoofer "Subwoofer power consumption  [%d W]" (GF_Living,GF_Energy) {zwave="26:1:command=meter,meter_scale=E_W,refresh_interval=60"} 
    Number Power_Powerbar_Reiceiver "Receiver power consumption  [%d W]" (GF_Living,GF_Energy) {zwave="26:2:command=meter,meter_scale=E_W,refresh_interval=70"} 
    Number Power_Powerbar_DVD "DVD power consumption [%d W]" (GF_Living,GF_Energy) {zwave="26:3:command=meter,meter_scale=E_W,refresh_interval=60"} 
    Number Power_Powerbar_TV "TV power consumption [%d W]" (GF_Living,GF_Energy) {zwave="26:4:command=meter,meter_scale=E_W,refresh_interval=70"} 
    Number Power_Powerbar_Xbox "XBOX-360 power consumption [%d W]" (GF_Living,GF_Energy) {zwave="26:5:command=meter,meter_scale=E_W,refresh_interval=80"} 
    Number Power_Powerbar_PC "Mediacenter power consumption [%d W]" (GF_Living,GF_Energy) {zwave="26:6:command=meter,meter_scale=E_W,refresh_interval=80"} 
    
    Number Energy_Powerbar_Subwoofer "Subwoofer total energy usage  [%.4f KWh]" (GF_Living) {zwave="26:1:command=meter,meter_scale=E_KWh,refresh_interval=300"} 
    Number Energy_Powerbar_Reiceiver "Receiver total energy usage  [%.4f KWh]" (GF_Living) {zwave="26:2:command=meter,meter_scale=E_KWh,refresh_interval=310"} 
    Number Energy_Powerbar_DVD "DVD totaal total energy usage  [%.4f KWh]" (GF_Living) {zwave="26:3:command=meter,meter_scale=E_KWh,refresh_interval=320"} 
    Number Energy_Powerbar_TV "TV total energy usage [%.4f KWh]" (GF_Living) {zwave="26:4:command=meter,meter_scale=E_KWh,refresh_interval=330"} 
    Number Energy_Powerbar_Xbox "XBOX-360 total energy usage  [%.4f KWh]" (GF_Living) {zwave="26:5:command=meter,meter_scale=E_KWh,refresh_interval=340"} 
    Number Energy_Powerbar_PC "Mediacenter total energy usage  [%.4f KWh]" (GF_Living) {zwave="26:6:command=meter,meter_scale=E_KWh,refresh_interval=350"} 
    
Honeywell Thermostat:

    /* Honeywell Thermostat with both heating and cooling and in Fahrenheit */

    Number Down_HVAC_HeatSetPoint "Heat Set [%.0f F]"	<thermostat>	(Group_HVAC_Downstairs)	{ zwave="7:command=thermostat_setpoint,setpoint_type=1,setpoint_scale=1" }
    Number Down_HVAC_CoolSetPoint "Cool Set [%.0f F]"	<thermostat>	(Group_HVAC_Downstairs)	{ zwave="7:command=thermostat_setpoint,setpoint_type=2,setpoint_scale=1" }
    Number Down_HVAC_Temperature  "Temperature [%.1f °F]" <thermostat> (Group_HVAC_Downstairs) { zwave="7:command=sensor_multilevel,sensor_type=1" }
    Number Down_HVAC_Mode "Mode [%d]"	(Group_HVAC_Downstairs)	{ zwave="7:command=thermostat_mode" }
    Number Down_HVAC_Fan_Mode "Fan Mode [%d]"	(Group_HVAC_Downstairs)	{ zwave="7:command=thermostat_fan_mode" }
    Number Down_HVAC_Operating_State "Opp State [MAP(thermostatOpState.map):%d]" (Group_HVAC_Downstairs) { zwave="7:command=thermostat_operating_state" }
    Number Down_HVAC_Fan_State "Fan State [MAP(thermostatFanState.map):%d]" (Group_HVAC_Downstairs)	{ zwave="7:command=thermostat_fan_state" }

monoprice motion detector (ZP3102)  

    Contact OfficeMotion            "office motion [MAP(motion.map):%s]"                               (ALL,motion,ff)              {zwave="2:command=BASIC"}
    Number  OfficeMotionBattery     "office motion battery [%d %%]"                                    (ALL,battery)                {zwave="2:command=BATTERY"}
    Contact OfficeMotionTamper      "office motion tamper [MAP(tamper.map):%s]"                        (ALL,tamper)                 {zwave="2:command=ALARM"}
    Number  OfficeMotionTemp        "office temp [%.1f °F]"                            <temperature>   (ALL,temperature,ff)         {zwave="2:command=sensor_multilevel,sensor_type=1,sensor_scale=1" }

Aeotec hidden door sensor (gen 5, DSB54)
  
    Contact garagewalkin_1_sensor    "garage walk-in [%s]"                                         (ALL,ff,sensor)              {zwave="3:command=BASIC"}
    Number  garagewalkin_1_battery   "garage walk-in battery [%d %%]"              <battery>       (ALL,battery)                {zwave="3:command=BATTERY"}

Ecolink PIR motion detector  

    Contact EntryMotion              "entry motion [MAP(motion.map):%s]"                           (ALL,motion,ff)              {zwave="4:command=SENSOR_BINARY"}
    Contact EntryMotionTamper        "entry motion tamper [MAP(tamper.map):%s]"    <battery>       (ALL,tamper)                 {zwave="4:command=ALARM"}
    Number  EntryMotionBattery       "entry motion battery [%d %%]"                <battery>       (ALL,battery)                {zwave="4:command=BATTERY"}

CT100 Thermostat  

    Number HVAC_HeatSetPoint        "Heat Set [%.1f F]"                            <thermostat>    (ALL,HVAC)                   {zwave="5:command=thermostat_setpoint,setpoint_type=1,setpoint_scale=1" }
    Number HVAC_CoolSetPoint        "Cool Set [%.1f F]"                            <thermostat>    (ALL,HVAC)                   {zwave="5:command=thermostat_setpoint,setpoint_type=2,setpoint_scale=1" }
    Number HVAC_Temperature         "Thermostat temperature [%.1f °F]"             <temperature>   (ALL,HVAC,ff)                {zwave="5:1:command=sensor_multilevel,sensor_type=1,refresh_interval=60"}
    Number HVAC_Humidity            "Thermostat humidity [%.1f %%]"                <humidity>      (ALL,HVAC,ff)                {zwave="5:2:command=sensor_multilevel,sensor_type=5,refresh_interval=60"}
    Number HVAC_Mode                "Mode [MAP(thermostatMode.map):%s]"            <climate>       (ALL,HVAC)                   {zwave="5:command=thermostat_mode"}
    Number HVAC_Fan_Mode            "Fan Mode [MAP(thermostatFanMode.map):%s]"                     (ALL,HVAC)                   {zwave="5:command=thermostat_fan_mode"}
    Number HVAC_Operating_State     "Operation State [MAP(thermostatOpState.map):%s]"              (ALL,HVAC)                   {zwave="5:command=thermostat_operating_state,refresh_interval=60"}
    Number HVAC_Fan_State           "Fan State [MAP(thermostatFanState.map):%s]"                   (ALL,HVAC)                   {zwave="5:command=thermostat_fan_state"}
    Number HVAC_Battery             "Thermostat battery [%d %%]"                   <battery>       (ALL,HVAC,battery)           {zwave="5:command=BATTERY"}

Aeotec Smart Switch 6 (ZW096)  

    Switch Smart6                   "Smart6 outlet"                                                (ff,ALL,outlet)              {zwave="6:command=switch_binary" }
    Number Smart6_Power             "Smart6 power  [%.2f W]"                                       (ALL,power)                  {zwave="6:command=meter,meter_scale=E_W" }
    Number Smart6_Energy            "Smart6 consumption  [%.2f KWh]"                               (ALL,power)                  {zwave="6:command=meter,meter_scale=E_KWh" }
    Number Smart6_Volts             "Smart6 voltage [%.2f V]"                                      (ALL,power)                  {zwave="6:command=meter,meter_scale=E_V"}
    Number Smart6_Amps              "Smart6 amperage [%.2f A]"                                     (ALL,power)                  {zwave="6:command=meter,meter_scale=E_A"}

Aeotec Door/Window Sensor (2nd Edition) Model: DSB29-ZWUS  

    Contact     zwave_contact_16_sensor "office - test door"                                             (doors,monitor)  {zwave="16:command=basic,respond_to_basic=true"}
    Contact     zwave_contact_16_tamper "office - test door tamper"                                      (doors,tamper)   {zwave="16:command=ALARM"}
    Number      FrontDoorBattery        "office - test door battery [%d %%]"              <battery>      (doors,battery)  {zwave="16:command=BATTERY"}

TBZ48 thermostat (does not have humidity sensor)  

    Number      HVAC_HeatSetPoint       "Heat Set [%.1f F]"                               <thermostat>   (HVAC)           {zwave="15:command=THERMOSTAT_SETPOINT,setpoint_type=1,setpoint_scale=1" }
    Number      HVAC_CoolSetPoint       "Cool Set [%.1f F]"                               <thermostat>   (HVAC)           {zwave="15:command=thermostat_setpoint,setpoint_type=2,setpoint_scale=1" }
    Number      HVAC_Temperature        "Thermostat temperature [%.1f °F]"                <temperature>  (HVAC)           {zwave="15:command=sensor_multilevel,sensor_type=1,refresh_interval=60"}
    Number      HVAC_Humidity           "Thermostat humidity [%.1f %%]"                                  (HVAC)           {zwave="15:command=sensor_multilevel,sensor_type=5,refresh_interval=60"}
    Number      HVAC_Mode               "Mode [MAP(thermostatMode.map):%s]"               <climate>      (HVAC)           {zwave="15:command=thermostat_mode"}
    Number      HVAC_Fan_Mode           "Fan Mode [MAP(thermostatFanMode.map):%s]"        <wind>         (HVAC)           {zwave="15:command=thermostat_fan_mode"}
    Number      HVAC_Operating_State    "Operation State [MAP(thermostatOpState.map):%s]" <climate>      (HVAC)           {zwave="15:command=thermostat_operating_state,refresh_interval=60"}
    Number      HVAC_Fan_State          "Fan State [MAP(thermostatFanState.map):%s]"      <wind>         (HVAC)           {zwave="15:command=thermostat_fan_state"}
    Number      HVAC_Battery            "HVAC battery state [%d %%]"                      <battery>      (HVAC,battery)	  {zwave="15:command=BATTERY"}

Fortrez water/temperature sensor  

    Number WaterTemp3 "Water Temp 3" (water,temp) {zwave="18:command=SENSOR_MULTILEVEL,sensor_type=1"}
    Contact Water3 "water sensor 3" (water) {zwave="18"}
    Number Water3_battery "Water 3 battery [%s %%]" (water) {zwave="18:command=BATTERY"}

Aeotec Multi Sensor 4  

    Number      Multi1_temp                "office - Multi Temperature [%.1f °F]"                  (multi,multiGraph) {zwave="10:command=SENSOR_MULTILEVEL,sensor_type=1"}
    Number      Multi1_humidity            "office - Multi Humidity    [%.0f %%]"                  (multi,multiGraph) {zwave="10:command=SENSOR_MULTILEVEL,sensor_type=5"}
    Number      Multi1_luminance           "office - Multi Luminance    [%.0f Lux]"                (multi)            {zwave="10:command=SENSOR_MULTILEVEL,sensor_type=3"}
    Contact     Multi1_motion              "office - Multi motion [MAP(motion.map):%s]" <motion>   (multi,motion)     {zwave="10:command=SENSOR_BINARY,respond_to_basic=true"}
    Number      Multi1_battery             "office - Multi battery [%d %%]"             <battery>  (multi,battery)    {zwave="10:command=BATTERY"}

Everspring Flood Detector model:ST812-2  

    Contact     zwave_water_9_sensor       "office - water sensor 2"                    <water>    (water,monitor)    {zwave="9:command=SENSOR_ALARM"}
    Number      Water_sensor_battery       "office - water sensor 2 battery [%d %%]"    <battery>  (water,battery)    {zwave="9:command=BATTERY"}

Fibaro Multisensor (FIB_FGMS-001)  

    Number  Movement         "Movement: [%s]"          <present>      { zwave="4:0:command=sensor_binary" }
    Number  Alarm            "Alarm: [%s]"             <fire>         { zwave="4:0:command=sensor_alarm" }
    Number  Lux              "Lux: [%.2f Lux]"         <sun>          { zwave="4:0:command=sensor_multilevel,sensor_type=3" }
    Number  Bat              "Battery: [%d %%]"        <energy>       { zwave="4:0:command=battery" }
    Number  Temp             "Temperature: [%.1f °C]"  <temperature>  { zwave="4:0:command=sensor_multilevel,sensor_type=1" }

Fibaro Wall Plug (FIB_FGWPF-101)  

    Switch Wall_Plug           "Wall Plug"                             { zwave="3:command=switch_binary"} 
    Number Wall_Plug_Power     "Wall Plug - current energy [%.1f W]"   { zwave="3:command=sensor_multilevel"}
    Number Wall_Plug_Energy    "Wall Plug - total energy [%.2f KWh]"   { zwave="3:command=meter" }

CT100 with refreshing setpoints (so manual changes are captured) items: 
  
    Number HVAC_HeatSetPoint "Heat Set [%.0f F]" <temperature> (gHVAC) {zwave="7:command=thermostat_setpoint,setpoint_type=1,setpoint_scale=1,refresh_interval=20" }
    Number HVAC_CoolSetPoint "Cool Set [%.0f F]" <temperature> (gHVAC) {zwave="7:command=thermostat_setpoint,setpoint_type=2,setpoint_scale=1,refresh_interval=20" }
    Number HVAC_Temperature "Temperature [%.1f F]" <temperature> (gHVAC) {zwave="7:1:command=sensor_multilevel,sensor_type=1,refresh_interval=60" }
    Number HVAC_Humidity "Rel Humidity [%.1f %%]" <temperature> (gHVAC) {zwave="7:2:command=sensor_multilevel,sensor_type=5,refresh_interval=60" }
    Number HVAC_Mode "Mode [MAP(thermostatMode.map):%d]" (gHVAC) {zwave="7:command=thermostat_mode" }
    Number HVAC_Fan_Mode "Fan Mode [MAP(thermostatFanMode.map):%d]" (gHVAC) {zwave="7:command=thermostat_fan_mode" }
    Number HVAC_Operating_State "Op State [MAP(thermostatOpState.map):%d]" (gHVAC) {zwave="7:command=thermostat_operating_state,refresh_interval=60" }
    Number HVAC_Fan_State "Fan State [MAP(thermostatFanState.map):%d]" (gHVAC) {zwave="7:command=thermostat_fan_state,refresh_interval=60" }
    Number HVAC_Battery "Battery State [%d %%]" (gHVAC) {zwave="7:command=BATTERY"}

Fibaro Universal-Dimmer (FGD-211)  

    Switch  swLight_HallCeiling      "Hall: Ceiling"          { zwave="9:command=SWITCH_MULTILEVEL" }
    Dimmer  diLight_HallCeiling      "Hall: Ceiling [%d %%]"  { zwave="9:command=SWITCH_MULTILEVEL" }
    Switch  swScene1_HallCeiling     "Hall-Simpleclick"       { zwave="9:command=SCENE_ACTIVATION,scene=26,state=0" }
    Switch  swScene2_HallCeiling     "Hall-Doubleclick"       { zwave="9:command=SCENE_ACTIVATION,scene=24,state=0" }
    Switch  swScene3_HallCeiling     "Hall-Tripleclick"       { zwave="9:command=SCENE_ACTIVATION,scene=25,state=0" }

Fibaro Door-Contact (FGK-101)  

    Contact   coStatus_DoorBell     "Doorbell: [MAP(bell.map):%s]"   { zwave="10:command=SENSOR_BINARY,respond_to_basic=TRUE" }
    Number    nuBatttery_DoorBell   "Doorbell: Battery [%s %%]"      { zwave="10:command=BATTERY" }

Fibaro Flood-Sensor (FGFS-101)  

    Contact coFibFlood_Alarm    "Water-Sensor: [MAP(water.map):%s]"   { zwave="11:command=SENSOR_ALARM, alarm_type=5,respond_to_basic=TRUE" }
    Number nuFibFlood_Battery   "Water-Sensor: Batterie [%s %%]"      { zwave="11:command=BATTERY" }

Smoke detector from Monoprice. They identify as Vision Security ZS6101.  

    Contact	iSmokeSensorMasterAlarm			"Master Status [%s]"		<fire>		(gSmokeSensorMaster)		{ zwave="8:command=SENSOR_BINARY,respond_to_basic=true" }
    Number	iSmokeSensorMasterBattery		"Master Battery [%s]"		<battery>	(gSmokeSensorMaster)		{ zwave="8:command=BATTERY" }

Fibaro RGBW Controller  

    Group	gWohnzimmer		"Wohnzimmer"			<sofa>	(gAlles)
    Group	gwzRGBW			"TV Rücklicht Erweitert"	<sofa>	(gWohnzimmer)

    Color 	wzRGBW 		"TV Rücklicht" 		<slider> 	(gwzRGBW)
    Dimmer 	wzRGBW_All 	"Helligkeit [%d %%]" 	<switch> 	(gwzRGBW)	{zwave="2"}
    Dimmer	wzRGBW_R 	"Rot [%d %%]" 		<switch> 	(gwzRGBW)	{zwave="2:2"}
    Dimmer 	wzRGBW_G 	"Grün [%d %%]" 		<switch> 	(gwzRGBW)	{zwave="2:3"}
    Dimmer 	wzRGBW_B 	"Blau [%d %%]" 		<switch> 	(gwzRGBW)	{zwave="2:4"}
    Dimmer 	wzRGBW_W 	"Weiß [%d %%]" 		<switch> 	(gwzRGBW)	{zwave="2:5"}
    
    Switch  wzRGBW_Switch	"Schalter Alle"		<switch> 	(gwzRGBW)	{ zwave="2:1"}
    Switch  wzRGBW_R_Switch	"Schalter Rot"		<switch> 	(gwzRGBW)	{ zwave="2:2"}
    Switch  wzRGBW_G_Switch "Schalter Grün"		<switch> 	(gwzRGBW)	{ zwave="2:3"}
    Switch  wzRGBW_B_Switch	"Schalter Blau"		<switch> 	(gwzRGBW)	{ zwave="2:4"}
    Switch  wzRGBW_W_Switch	"Schalter Weiß"         <switch> 	(gwzRGBW)	{ zwave="2:5"}

    Number wzRGBW_Power     "Stromverbrauch [%.1f W]"   	<energy> 	(gwzRGBW)	{ zwave="2:command=sensor_multilevel"}
    Number wzRGBW_Energy    "Gesamtverbrauch [%.2f KWh]"   	<energy> 	(gwzRGBW)	{ zwave="2:command=meter", refresh_interval=60}

Fibaro Rollershutter FGRM-222  

    Group	gKind			"Kinderzimmer"				<boy3>		(gAlles)
    Group	gkdSHUTTER		"Kinderzimmer Außenrollo Erweitert"	<rollershutter>	(gKind)

    Rollershutter kdSHUTTER		"Außenrollo [%d %%]" 				(gkdSHUTTER)	{zwave="3:command=switch_multilevel,invert_state=false,invert_percent=true"}

    Number kdSHUTTER_Power		"Stromverbrauch [%.1f W]"	<energy> 	(gkdSHUTTER)	{ zwave="3:command=sensor_multilevel"}
    Number kdSHUTTER_Energy		"Gesamtverbrauch [%.2f KWh]"	<energy> 	(gkdSHUTTER)	{ zwave="3:command=meter" }

Fibaro Smoke detector  

    Contact	Z_Kitchen_Smoke	"Smokedetector_K is[%s]"	(Smoke_Alarm) {zwave="6:command=sensor_alarm,alarm_type=1" }
    Contact	Z_Kitchen_Tamper	"Smoke_sensor_K Tamper is[MAP(en.map):%s]"	(Tamper_Alarm)	{ zwave="6:command=sensor_alarm,alarm_type=0" }
    Number	Z_Kitchen_Battery	"Smoke_sensor_batt [%d %%]"	(Battery_Levels)	{zwave="6:command=battery" }
    Number	Z_Kitchen_Temp	"Kitchen_temperature [%.1f°C]"	(Temperatures) {zwave="6:command=sensor_multilevel,sensor_type=1" }

Fibaro Universal Sensor  

    Contact	Z_deurbel0	"Z_deurbel_sensor"	{ zwave="8:0:command=SENSOR_BINARY,respond_to_basic=TRUE" }
    Contact	Z_deurbel1	"Z_deurbel_input1"	{ zwave="8:1:command=SENSOR_BINARY,respond_to_basic=TRUE" }
    Contact	Z_deurbel2	"deurbel"	{ zwave="8:2:command=SENSOR_BINARY,respond_to_basic=TRUE" }

Z-wave.me double paddle wall switch  

    Switch WCD1_1_BUT1 "Test BUT 1" { zwave="7:command=SCENE_ACTIVATION,scene=11,state=1" }
    Switch WCD1_1_BUT3 "Test BUT 3" { zwave="7:command=SCENE_ACTIVATION,scene=12,state=0" }
    Switch WCD1_1_BUT2 "Test BUT 2" { zwave="7:command=SCENE_ACTIVATION,scene=21,state=1" }
    Switch WCD1_1_BUT4 "Test BUT 4" { zwave="7:command=SCENE_ACTIVATION,scene=22,state=0" }
    Switch WCD1_1_SW1 "Test WCD SW 1" (gTest) 
    Switch WCD1_1_SW2 "Test WCD SW 2" (gTest)

Heat-it thermostat  

    Number Temperature { zwave="4:0:command=SENSOR_MULTILEVEL,sensor_type=1" }
    Number Set_Temp { zwave="4:command=THERMOSTAT_SETPOINT,setpoint_type=1,setpoint_scale=0" }
    Number Mode {zwave="4:0:command=THERMOSTAT_MODE" }
    DateTime LastUpdated { zwave="4:command=info,item=LAST_UPDATE"}

Everspring Wireless Dimmer ADA1311  

    Dimmer Light { zwave="6:command=SWITCH_MULTILEVEL,respond_to_basic=true" }

Z-Weather Weather Station  

    Number  Windspeed       "Wind [%.2f m/s]"       <wind>  (weather_station)               {     zwave="5:command=sensor_multilevel,sensor_type=6,refresh_interval=300" }
    Number  Luminance       "Luminance [%.1f %%]"   (weather_station)               { zwave="5:command=sensor_multilevel,sensor_type=3,refresh_interval=300" }
    Number  RelativeHumidity        "Humidity [%.1f %%]"    (weather_station)               { zwave="5:command=sensor_multilevel,sensor_type=5,refresh_interval=300" }
    Number  DewPoint        "Dew Point [%.1f °C]"   (weather_station)               { zwave="5:command=sensor_multilevel,sensor_type=11,refresh_interval=300" }
    Number  BarometricPressure      "Barometric Pressure [%.1f kPa]"        (weather_station)               { zwave="5:command=sensor_multilevel,sensor_type=9,refresh_interval=300" }
    Number  TempWeatherStation      "Temp Weatherstation [%.1f °C]" <temperature>   (weather_station)               { zwave="5:command=sensor_multilevel,sensor_type=1,refresh_interval=300" }
    Number  BatteryWeatherStation   "Battery Weatherstation [%.2f %%]"      { zwave="5:command=battery,refresh_interval=600" }

Fibaro FGK-101 door sensor (requires DS18B20 to be added):  

    Number  Temp_UtilityRoom "Utility room temperature [%.1f °C]" { zwave="7:2:command=SENSOR_MULTILEVEL" }

ecolink tiltzwave1 sensor for garage door.  

    Group DoorsWindows "Doors and Windows"
    Contact GarageDoor "Garage Door is [MAP(en.map):%s]" (DoorsWindows){zwave="3:command=SENSOR_BINARY"}

Fibaro Universal Sensor with attached DS18B20 temperature sensors:  

    Number Sensor_Temp_1 "Temp1 [%.2f °C]" { zwave="17:3:command=sensor_multilevel" }
    Number Sensor_Temp_2 "Temp1 [%.2f °C]" { zwave="17:4:command=sensor_multilevel" }
    Number Sensor_Temp_3 "Temp1 [%.2f °C]" { zwave="17:5:command=sensor_multilevel" }

Nodon CRC 3100 (Octan Remote):  

1.- Change the configuration parameter 3 to 1.
2.- Create the items as shown below:

    Switch	Nodon_Button1_Single	"Nodon_Button1_Single"	<switch>	{ zwave="8:command=SCENE_ACTIVATION,scene=10,state=1" }
    Switch	Nodon_Button2_Single	"Nodon_Button2_Single"	<switch>	{ zwave="8:command=SCENE_ACTIVATION,scene=20,state=1" }
    Switch	Nodon_Button3_Single	"Nodon_Button3_Single"	<switch>	{ zwave="8:command=SCENE_ACTIVATION,scene=30,state=1" }
    Switch	Nodon_Button4_Single	"Nodon_Button4_Single"	<switch>	{ zwave="8:command=SCENE_ACTIVATION,scene=40,state=1" }

    Switch  Nodon_Button1_Long      "Nodon_Button1_Long"  <switch>          { zwave="8:command=SCENE_ACTIVATION,scene=12,state=1" }
    Switch  Nodon_Button2_Long      "Nodon_Button2_Long"  <switch>          { zwave="8:command=SCENE_ACTIVATION,scene=22,state=1" }
    Switch  Nodon_Button3_Long      "Nodon_Button3_Long"  <switch>          { zwave="8:command=SCENE_ACTIVATION,scene=32,state=1" }
    Switch  Nodon_Button4_Long      "Nodon_Button4_Long"  <switch>          { zwave="8:command=SCENE_ACTIVATION,scene=42,state=1" }

    Switch  Nodon_Button1_Release   "Nodon_Button1_Release" <switch>        { zwave="8:command=SCENE_ACTIVATION,scene=11,state=1" }
    Switch  Nodon_Button2_Release   "Nodon_Button2_Release" <switch>        { zwave="8:command=SCENE_ACTIVATION,scene=21,state=1" }
    Switch  Nodon_Button3_Release   "Nodon_Button3_Release" <switch>        { zwave="8:command=SCENE_ACTIVATION,scene=31,state=1" }
    Switch  Nodon_Button4_Release   "Nodon_Button4_Release" <switch>        { zwave="8:command=SCENE_ACTIVATION,scene=41,state=1" }

    Switch  Nodon_Button1_Double    "Nodon_Button1_Double" <switch>         { zwave="8:command=SCENE_ACTIVATION,scene=13,state=1" }
    Switch  Nodon_Button2_Double    "Nodon_Button2_Double" <switch>         { zwave="8:command=SCENE_ACTIVATION,scene=23,state=1" }
    Switch  Nodon_Button3_Double    "Nodon_Button3_Double" <switch>         { zwave="8:command=SCENE_ACTIVATION,scene=33,state=1" }
    Switch  Nodon_Button4_Double    "Nodon_Button4_Double" <switch>         { zwave="8:command=SCENE_ACTIVATION,scene=43,state=1" }

aeotec multisensor 6  

    Contact Motion_GFToilet "Motion [MAP(motion.map):%s]" (GF_Toilet) { zwave="8:command=sensor_binary,respond_to_basic=true" }
    Number Alarm_GFToilet "Alarm: [%s]" (GF_Toilet) { zwave="8:command=alarm" }
    Number Temp_GFToilet "Temperature: [%.1f °C]" (GF_Toilet) { zwave="8:command=sensor_multilevel,sensor_type=1,sensor_scale=0" }
    Number Humid_GFToilet "Humidity: [%.0f %%]" (GF_Toilet) { zwave="8:command=sensor_multilevel,sensor_type=5" }
    Number Lumin_GFToilet "Luminance: [%.0f Lux]" (GF_Toilet) { zwave="8:command=sensor_multilevel,sensor_type=3" }
    Number Battery_GFToilet "Battery: [%d %%]" (GF_Toilet) { zwave="8:command=battery" }

Eurotronic Stella Z thermostat.  

    Number Temp_Sensor_StellaZ_Bad "Badezimmer Temperatur: [%.1f C]" <temperature> (Heizung,Bad,Temperaturen) { zwave="28:command=sensor_multilevel,sensor_type=1" }
    Number Battery_Sensor_StellaZ_Bad "Badezimmer Batterie: [%d %%]" <energy> (Heizung,Batterien) { zwave="28:command=battery" }
    Number Temp_Setpoint_StellaZ_Bad " [%d]" <temperature> (Heizung,Heizung_Soll) { zwave="28:command=thermostat_setpoint, setpoint_type=1" }

D-Link DCH-Z110  

    Contact	C	"Movement [%s]"	(gf)	{ zwave="10:command=SENSOR_BINARY,respond_to_basic=TRUE" }
    Number	T	"Temperature [%.1f °C]"	(gf)	{ zwave="10:command=sensor_multilevel,sensor_type=1" }
    Number	H	"Luminance [%.0f Lux]"	(gf)	{ zwave="10:command=sensor_multilevel,sensor_type=3" }
