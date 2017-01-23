The [AlarmDecoder](http://www.alarmdecoder.com) is a hardware adapter that interfaces with Ademco/Honeywell alarm panels. It acts
essentially like a keypad, reading and writing messages on a serial bus that connects keypads with the main panel.

There are several versions of the adapter available: ad2pi (a board that plugs into a raspberry pi and so offers network-based tcp connectivity),
ad2serial (serial port access), or ad2usb (emulated serial port via usb).

This binding allows openhab to access the status of contacts and motion detectors connected to Honeywell Vista 20p and similar alarm panels.

## Hardware setup and preparation

How to wire the alarm decoder into the panel is described in the alarm
decoder quickstart guide.  Before
working on the main panel it is advisable to put the alarm
system in test mode, and unplug the phone connection to it for good
measure (don't forget to plug it back in when finished).

Understanding exactly what expansion boards are connected to the main
panel is crucial for a successful setup of the alarmdecoder and also helpful in interpreting
the messages from the alarmdecoder.

While many of the expansion devices don't have labels on the outside,
inserting a flat screwdriver into the right slot and prying gently
will usually uncover a circuit board with numbers on it that can be web
searched.

Although not mentioned in the quickstart guide, and only documented
on an odd
[thread](http://archive.nutech.com/index.php?option=com_fireboard&Itemid=74&func=view&catid=4&id=656),
configuring the virtual relay boards is absolutely necessary on panels
like the Honeywell Vista 20p and similar, or else all of the eight
on-board zones will not be visible! The process sounds intimidating,
and it does require bypassing the installer code (see
panel documentation about that), but it is not all that hard. 

Once the hardware has been set up properly, a simple, [well
documented](http://www.alarmdecoder.com/wiki/index.php/Protocol)
clear text ASCII byte stream is obtained, either on a serial port,
or (with the ad2pi appliance) a tcp port that can be telneted to.

Here is an example ASCII stream straight from the alarmdecoder:
```
   !SER2SOCK Connected
   !SER2SOCK SERIAL_CONNECTED
   [0000000110000000----],005,[f70000ff1005000028020000000000],"FAULT 05 MUSIC  ROOM WINDOW     "
   !REL:14,02,01
   [0000030110000000----],006,[f70000ff1006030028020000000000],"FAULT 06 OFFICE WINDOW          "
   !REL:14,02,00
   [0000000110000000----],014,[f70000ff1014000028020000000000],"FAULT 14 KITCHENDOOR            "
   !RFX:0717496,80
   !RFX:0717496,00
   !RFX:0610922,80
   !RFX:0610922,00
   [0000000110000000----],014,[f70000ff1014000028020000000000],"FAULT 14 KITCHENDOOR            "
```
Each alarm zone of the panel is represented by a unique combination of
message *type* and *address*.
The message *type* depends on how the zone is connected to the panel:
via radio frequency (RFX), a zone expander board (EXP), a relay
board (REL), or as a keypad (KPM). For instance: !REL:14,02,01 indicates that relay board
14, channel 02 has gone into state 01. The message *type* is REL, the
*address* is 14,02.

Before configuring the binding one must determine which zone generates what
message. The easiest way is to observe the ascii stream while 
faulting a given zone by e.g. opening a window or door.

## Configuring openhab

The openhab.cfg file allows to configure either a tcp:
```
alarmdecoder:connect=tcp:ad2pihostname.mydomain.com:port
```
or a serial connection:
```
alarmdecoder:connect=serial:/dev/ttyUSB0
```
Warning: using an alarmdecoder via serial port has not been debugged yet!
To send commands from openhab to the alarm panel, add this configuration line:
```
alarmdecoder:send_commands_and_compromise_security=true
```
Once this option is set to true, the alarm system can potentially be disabled from openHAB.
You have been warned!

Once the openhab.cfg is taken care of, create a suitable file
(e.g. alarmdecoder.items) in the items folder of your openhab home
directory.

Here is an example file that instantiates some Number and
Contact items, and a String item
```
Group gPanel "alarm panel" (All)

Number alarmPanelStatusRaw	    "panel status: [%d]" (gPanel)  	  {alarmdecoder="KPM:00#status"}
Number alarmPanelStatusReady	    "panel ready: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#status,bit=17"}
Number alarmPanelStatusAway	    "panel away: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#status,bit=16"}
Number alarmPanelStatusHome	    "panel home: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#status,bit=15"}
Number alarmPanelStatusBacklight    "panel backlight: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#status,bit=14"}
Number alarmPanelStatusProgramming  "panel programming: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#status,bit=13"}

Number alarmPanelStatusBypass	    "panel bypassed: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#status,bit=9"}
Number alarmPanelStatusPower	    "panel on AC: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#status,bit=8"}
Number alarmPanelStatusChime	    "panel chime: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#status,bit=7"}
Number alarmPanelStatusAlarmOccured "panel alarm occurred: [%d]" (gPanel)  {alarmdecoder="KPM:00#status,bit=6"}
Number alarmPanelStatusAlarm	    "panel alarm sounding: [%d]" (gPanel)  {alarmdecoder="KPM:00#status,bit=5"}
Number alarmPanelStatusBatteryLow   "panel battery low: [%d]" (gPanel)     {alarmdecoder="KPM:00#status,bit=4"}
Number alarmPanelStatusDelay	    "panel delay off: [%d]" (gPanel)  	  {alarmdecoder="KPM:00#status,bit=3"}
Number alarmPanelStatusFire	    "panel fire: [%d]" (gPanel)  	  {alarmdecoder="KPM:00#status,bit=2"}
Number alarmPanelStatusZoneIssue    "panel zone issue: [%d]" (gPanel)  	  {alarmdecoder="KPM:00#status,bit=1"}
Number alarmPanelStatusArmedStay    "panel armed stay: [%d]" (gPanel)  	  {alarmdecoder="KPM:00#status,bit=0"}

Contact alarmPanelContactReady	    "panel ready: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#contact,bit=17"}
Contact alarmPanelContactAway	    "panel away: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#contact,bit=16"}
Contact alarmPanelContactHome	    "panel home: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#contact,bit=15"}
Contact alarmPanelContactBacklight    "panel backlight: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#contact,bit=14"}
Contact alarmPanelContactProgramming  "panel programming: [%d]" (gPanel)   {alarmdecoder="KPM:00#contact,bit=13"}
Contact alarmPanelContactBypass	    "panel bypassed: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#contact,bit=9"}
Contact alarmPanelContactPower	    "panel on AC: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#contact,bit=8"}
Contact alarmPanelContactChime	    "panel chime: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#contact,bit=7"}
Contact alarmPanelContactAlarmOccured "panel alarm occurred: [%d]" (gPanel)  {alarmdecoder="KPM:00#contact,bit=6"}
Contact alarmPanelContactAlarm	    "panel alarm sounding: [%d]" (gPanel)  {alarmdecoder="KPM:00#contact,bit=5"}
Contact alarmPanelContactBatteryLow   "panel battery low: [%d]" (gPanel)     {alarmdecoder="KPM:00#contact,bit=4"}
Contact alarmPanelContactDelay	    "panel delay off: [%d]" (gPanel)  	  {alarmdecoder="KPM:00#contact,bit=3"}
Contact alarmPanelContactFire	    "panel fire: [%d]" (gPanel)  	  {alarmdecoder="KPM:00#contact,bit=2"}
Contact alarmPanelContactZoneIssue    "panel zone issue: [%d]" (gPanel)    {alarmdecoder="KPM:00#contact,bit=1"}
Contact alarmPanelContactArmedStay    "panel armed stay: [%d]" (gPanel)    {alarmdecoder="KPM:00#contact,bit=0"}

Number alarmPanelBeeps	    	    "panel beeps: [%d]" (gPanel) 	  {alarmdecoder="KPM:00#beeps"}
Number alarmPanelZone	    	    "panel zone: [%d]" (gPanel) 	  	  {alarmdecoder="KPM:00#zone"}
String alarmPanelDisplay	    "panel display: [%s]" (gPanel) 	  {alarmdecoder="KPM:00#text"}
```
Note that the status bits are accessible as either Contacts or
Numbers. All but a few of them are quite useless.

Here is how to bind items to RFX, REL, and EXP messages:
```
Group gContact "contacts" (All)
Group gNumber "data" (All)
Contact zone2  	       "zone 2 [MAP(contact.map):%s]"   (gContact) {alarmdecoder="EXP:07,08#contact"}
Contact zone1  	       "zone 1 [MAP(contact.map):%s]"  (gContact) {alarmdecoder="REL:13,01#contact"}
Contact motionContact  "motion sensor contact [MAP(contact.map):%s]" (gContact) {alarmdecoder="RFX:0923844#contact,bitmask=0x80"}
Contact WindowContact  "window/door magnetic contact (5816) [MAP(contact.map):%s]" (gContact) {alarmdecoder="RFX:0923844#contact,bitmask=0x20"}
Number  motionData     "motion sensor data [%d]"	  (gNumber)  {alarmdecoder="RFX:0923844#data"}
Number  motionLowBattery       "motion sensor battery [%d]" (gNumber) {alarmdecoder="RFX:0923844#data,bit=1"}
Number  motionNeedsSupervision "motion sensor supervision [%d]" (gNumber) {alarmdecoder="RFX:0923844#data,bit=2"}
```

Just like for the KPM messages, the RFX messages are exposed either as
a Number item, or as a Contact. Since the REL and EXP messages just give
binary data, they are only mapped to contact items.

```
Bitmask values for RFX messages are as follows:
  0x02 - battery alert
  0x04 - supervision (tamper)
  0x10 - loop3
  0x20 - loop2
  0x40 - loop4
  0x80 - loop1
```

Most wireless devices will transmit their state on loop1 (0x80).    Honeywell 5816 sensors (often used on doors/windows) use loop1 (0x80) for external contacts, and loop2 (0x40) for the magnetic contact; the 5804 remote keyfob uses one zone for each button.

If you don't care much about security and want to operate your alarm keypad from within openhab,
enable ```send_commands_and_compromise_security=true``` as discussed above, and add these lines to
your alarmdecoder.items file:

    Number alarmPanelLine1 "" (gPanel)	 {alarmdecoder="SEND#1=1,2=2,3=3", autoupdate="false"}
    Number alarmPanelLine2 "" (gPanel)	 {alarmdecoder="SEND#4=4,5=5,6=6", autoupdate="false"}
    Number alarmPanelLine3 "" (gPanel)	 {alarmdecoder="SEND#7=7,8=8,9=9", autoupdate="false"}
    Number alarmPanelLine4 "" (gPanel)	 {alarmdecoder="SEND#10=*,0=0,11=POUND", autoupdate="false"}

    String sAlarmPanelDisplay "panel display: [%s]" (gPanel) 	  {alarmdecoder="KPM:00#text"}

These items accept "Number" commands from the openHAB bus, and map them to strings which are then SEND to the alarm panel. In the above example, when a number 10 arrives, a "*" is sent to the alarmdecoder, or any string that is configured. Note that the special character sequence POUND will be further translated to "#".

For the GUI to show the keypad, the following lines need to be added to the sitemap file:

    Switch item=alarmPanelLine1 label="line1" mappings=[ 1="1____(OFF)",  2="2(AWAY)",  3="3__(STAY)"]
    Switch item=alarmPanelLine2 label="line2" mappings=[ 4="____4_____",  5="5(TEST)",  6="6(BYPASS)"]
    Switch item=alarmPanelLine3 label="line3" mappings=[ 7="7(INSTANT)",  8="8(CODE)",  9="9_(CHIME)"]
    Switch item=alarmPanelLine4 label="line4" mappings=[10="*__(READY)",  0="___0___", 11="____#____"]
    Text item=sAlarmPanelDisplay

## Long Range Radio (LRR) Messages (since Release 1.10.0)
Starting from Release 1.10.0, Long Range Radio (LRR) Messages are also supported.
LRR must be enabled in your Alarm Panel. For Vista series (or other Honeywell/Ademco) panels, this can be configured in field *29

Overview of LRR Support : http://www.alarmdecoder.com/wiki/index.php/LRR_Support

Alarm Decoder LRR Protocol : http://www.alarmdecoder.com/wiki/index.php/Protocol#LRR

LRR format from Alarm Decoder is:

```
!LRR:<data>,<partition>,<event>
```

Because ```<data>``` could be anything, it cannot be tied to a particular item as opposed to item bindings such as Contacts mentioned above. Instead, the entire message is sent to the binding.

```
!LRR:<entiremessage>
```

Example usage for LRR:

**ITEMS:**
```
String sAlarmLRR "LLR Msg: [%s]" (gAlarmPanel) {alarmdecoder="LRR:00#text"}
String sAlarmLRR_lastUser "Last User: [%s]" (gAlarmPanel)
String sAlarmLRR_lastEvent "Last Event: [%s]" (gAlarmPanel)
DateTime sAlarmLRR_lastEventDateTime ""  (gAlarmPanel)
String sAlarmLRR_lastPartition "Last Partition: [%s]" (gAlarmPanel)
```

**RULE (parsing the message):**
```
import java.util.Map;
import java.util.HashMap;
import com.google.common.collect.ImmutableMap

val Map<String, String> UserList = ImmutableMap.<String, String>builder()
    .put("001", "Installer")
    .put("002", "Master")
    .put("003", "User1")
    .put("004", "User2")
    .build()

rule "LRR Parser"
when
	Item sAlarmLRR received update
then
	var String msg = sAlarmLRR.state.toString();
	var String[] parts = msg.split(",");
	if (parts.length != 3) {
		logError("rules", "sAlarmLRR rule parsing error.");
	} else {
		logInfo("rules", "Parsing sAlarmLRR==> " + msg);
		var String user = UserList.get(parts.get(0));	
		var String partition = parts.get(1);
		var String event = transform("MAP", "alarm_LRR_eventmap.map", parts.get(2).toString);
		
		postUpdate(sAlarmLRR_lastUser, user);
		postUpdate(sAlarmLRR_lastPartition, partition);
		postUpdate(sAlarmLRR_lastEvent, event);
		postUpdate(sAlarmLRR_lastEventDateTime, new DateTimeType());
	}
end
```

**Transformation (alarm_LRR_eventmap.map):**
```
ALARM_EXIT_ERROR=Zone not closed during arming
TROUBLE=Tamper or failure
BYPASS=Zone was bypassed
ACLOSS=AC power was lost
LOWBAT=Low battery indication
TEST_CALL=Testing mode
OPEN=Alarm disarmed
ARM_AWAY=Armed AWAY
ARM_STAY=Armed STAY
RFLOWBAT=RF Low battery
CANCEL=Alarm canceled second disarm
RESTORE=Alarm restored
TROUBLE_RESTORE=Trouble event cleared
BYPASS_RESTORE=Bypassed zone cleared
AC_RESTORE=AC power restored
LOWBAT_RESTORE=Low battery restored
RFLOWBAT_RESTORE=RF Low battery cleared
TEST_RESTORE=Testing Mode cleared
ALARM_PANIC=There was a panic
ALARM_FIRE=There was a fire
ALARM_ENTRY=There was an entry alarm
ALARM_AUX=Auxiliary alarm triggered
ALARM_AUDIBLE=Audible alarm
ALARM_SILENT=Silent alarm
ALARM_PERIMETER=Perimeter alarm
NULL=Unknown

```

## Trouble shooting and debugging

To get additional debugging information, insert the following into
your `logback.xml` file:
```
    <appender name="ALARMDECODERFILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/alarmdecoder.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>logs/alarmdecoder-%d{yyyy-ww}.log.zip</fileNamePattern>
        <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
                <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{30}[:%line]- %msg%n%ex{5}</pattern>
        </encoder>
    </appender>
    <!-- Change DEBUG->TRACE for even more detailed logging -->
    <logger name="org.openhab.binding.alarmdecoder" level="DEBUG" additivity="false">
    <appender-ref ref="ALARMDECODERFILE" />
    </logger>
```
This will log additional debugging messages to a separate file in the
log directory.

## Quirks

1. The alarmdecoder cannot query the panel for the state of individual
zones. For this reason, the binding puts contacts into the "unknown"
state, *until the panel goes into the READY state*. At that point all
contacts for which no messages have arrived are presumed to be in the
CLOSED state. In other words: to get to a clean slate after an openhab restart,
close all doors/windows such that the panel is READY.

2. The sitemap provided above may not display properly when viewing from the classical GUI. It has been confirmed to work with GreenT, Habdroid and IOS.