This page is an attempt to creating a complete working example of CT100 Thermostat support.
I found the parent page only had items so was missing the needed transforms and rules.
I also did not want to accidentally break formatting for other entries.

# Items

## items/CT100.items

    Group gHVAC "Heat Pump" (All)
    
    Number HVAC_HeatSetPoint "Heat Set [%.0f F]" <temperature> (gHVAC) {zwave="2:command=thermostat_setpoint,setpoint_type=1,setpoint_scale=1,refresh_interval=20" }
    Number HVAC_CoolSetPoint "Cool Set [%.0f F]" <temperature> (gHVAC) {zwave="2:command=thermostat_setpoint,setpoint_type=2,setpoint_scale=1,refresh_interval=20" }
    Number HVAC_Temperature "Temperature [%.1f F]" <temperature> (gHVAC) {zwave="2:1:command=sensor_multilevel,sensor_type=1,refresh_interval=60" }
    Number HVAC_Humidity "Rel Humidity [%.1f %%]" <temperature> (gHVAC) {zwave="2:2:command=sensor_multilevel,sensor_type=5,refresh_interval=60" }
    Number HVAC_Mode "Mode [MAP(thermostatMode.map):%d]" (gHVAC) {zwave="2:command=thermostat_mode,refresh_interval=60" }
    Number HVAC_Fan_Mode "Fan Mode [MAP(thermostatFanMode.map):%d]" (gHVAC) {zwave="2:command=thermostat_fan_mode,refresh_interval=60" }
    Number HVAC_Operating_State "Op State [MAP(thermostatOpState.map):%d]" (gHVAC) {zwave="2:command=thermostat_operating_state,refresh_interval=20" }
    Number HVAC_Fan_State "Fan State [MAP(thermostatFanState.map):%d]" (gHVAC) {zwave="2:command=thermostat_fan_state,refresh_interval=20" }
    Number HVAC_Battery "Battery State [%d %%]" (gHVAC) {zwave="2:command=BATTERY,refresh_interval=300"}

Note: In My system the CT100 was the first device (after the Z-Stick) so it is NODE 2, in your system you will need to update the NODE to match your configuration. (zwave="**2**:command...)

# Transform files

You will notice that the above items reference 4 different map files. I did find 2 of them mentioned in forum posts but the others I have made up as I go...

## transform/thermostatMode.map
    -1=Unintialized
    0=Off
    1=Heat
    2=Cool
    3=Auto
    4=Aux Heat
    5=Resume
    6=Fan Only
    7=Furnace
    8=Dry Air
    9=Moist Air
    10=Auto Changeover
    11=Heat Eco
    12=Cool Eco
    13=Away

## transform/thermostatOpState.map 
    -=Uninitialized
    0=Idle
    1=Heating
    2=Cooling
    3=Fan Only
    4=Pending Heat
    5=Pending Cool
    6=Vent / Economizer

## transform/thermostatFanMode.map 
    -=Uninitialized
    0=Auto
    1=On
    2=Two
    3=Three
    4=Four
    5=Five
    6=Six

I am fairly sure only 0 and 1 are used, the extra ones are just fillers in case I am wrong.
I could not find any documentation verifying this.

## transform/thermostatFanState.map 
    -=Uninitialized
    0=Off
    1=On
    2=Two
    3=Three
    4=Four
    5=Five
    6=Six

Again, I am fairly sure only 0 and 1 are used.

# Rules

So far I only have a rule that turns on the Fan if the system has been idle for an hour. A simple effort to keep the air an even temperature/humidity in the house. Here in Florida this will typically only happen late at night.

    import org.openhab.model.script.actions.Timer
    
    val String PATTERN = "HH:mm "
    var Timer timerFanIdle = null
    var Timer timerFanCirculate = null
    var Integer ruleRunning = 0
    
    rule "Idle Fan"
    when
        Item HVAC_Fan_State changed to 0
    then
        var String ts = now.toString(PATTERN)
        if (timerFanIdle != null) {
          timerFanIdle.cancel
          timerFanIdle = null
        }
        if (timerFanCirculate != null) {
          timerFanCirculate.cancel
          timerFanCirculate = null
        }
        timerFanIdle = createTimer(now.plusMinutes(60)) [|
            ts = now.toString(PATTERN)
            ruleRunning = 1
      	    sendCommand(HVAC_Fan_Mode, 1)
            sendBroadcastNotification(ts + "Run Fan to circulate")
            timerFanCirculate = createTimer(now.plusMinutes(15)) [|
                    ts = now.toString(PATTERN)
	          	sendCommand(HVAC_Fan_Mode, 0)
            	ruleRunning = 0 ]
            ]
         }
    end

    rule "Running Fan"
    when
        Item HVAC_Fan_State changed to 1
    then
        var String ts = now.toString(PATTERN)
        if (ruleRunning == 0) {
          if (timerFanIdle != null) {
            timerFanIdle.cancel
            timerFanIdle = null
          }
          if (timerFanCirculate != null) {
            timerFanCirculate.cancel
            timerFanCirculate = null
          }
        }
