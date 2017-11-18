# Fritzbox Binding using TR064 protocol


This is a binding for communication with AVM Fritz!Box using SOAP requests (TR064 protocol). I tested it on

* 7270
* 7360SL (v6.30)
* 7390
* 6360 Cable (v6.04)
* 7490

**Features:**
* detect if MAC is online in network (presence detection)
* switching on/off 2,4Hz Wifi, 5GHz Wifi and Guest Wifi (if any)
* getting external IP address of fbox
* getting fbox model name
* call monitor
 * Switch Item: Receives "ON" state when call is incoming
 * Call Items: Shows external an internal number for incoming/outgoing calls
 * Resolve external call number to phonebook name
* enabling/disabling telephone answering machines (TAMs) 
* getting new messages per TAM
* getting missed calls for the last x days

**Prerequisites**

Java JRE 1.8.0+ (version 1.7 / openJDK7 is not working)

## Download
Until I manage it to integrate the binding in the build process, you may download the latest version [here] (https://github.com/gitbock/fritzboxtr064/releases)

## Installation
Put the *.jar file into your openHAB "addons" directory.

## Configuration

### FritzBox
* Enable TR064: In the webui goto "Heimnetz" - "Netzwerkeinstellungen": enable option "Zugriff für Anwendungen zulassen" (enabled by default)
* Only if you want to use the call monitor feature (items starting with callmonitor_...), enable the interface by dialing #96\*5\* You may disable it again by dialing #96\*4\*

### Item Binding
```
String  fboxName            "FBox Model [%s]"           {fritzboxtr064="modelName"}
String  fboxWanIP           "FBox WAN IP [%s]"          {fritzboxtr064="wanip"}
Switch  fboxWifi24          "2,4GHz Wifi"               {fritzboxtr064="wifi24Switch"}
Switch  fboxWifi50          "5,0GHz Wifi"               {fritzboxtr064="wifi50Switch"}
Switch  fboxGuestWifi       "Guest Wifi"                {fritzboxtr064="wifiGuestSwitch"}
Contact cFboxMacOnline      "Presence (WiFi) [%s]"      {fritzboxtr064="maconline:11-11-11-11-11-11" }

# only when using call monitor
Switch  fboxRinging	 	  	"Phone ringing [%s]"                {fritzboxtr064="callmonitor_ringing" }
Switch  fboxRinging_Out	 	  	"Phone ringing [%s]"                {fritzboxtr064="callmonitor_outgoing" }
Call    fboxIncomingCall   	"Incoming call: [%1$s to %2$s]"     {fritzboxtr064="callmonitor_ringing" } 
Call    fboxOutgoingCall    "Outgoing call: [%1$s to %2$s]"     {fritzboxtr064="callmonitor_outgoing" }

# resolve numbers to names according phonebook
Call    fboxIncomingCallResolved   	"Incoming call: [%1$s to %2$s]"     {fritzboxtr064="callmonitor_ringing:resolveName" } 

# Telephone answering machine (TAM) items
# Number after tamSwitch is ID of configured TAM, start with 0
Switch  fboxTAM0Switch   "Answering machine ID 0"		{fritzboxtr064="tamSwitch:0"}
Number  fboxTAM0NewMsg   "New Messages TAM 0 [%s]"      {fritzboxtr064="tamNewMessages:0"}

# Missed calls: specify the number of last days which should be searched for missed calls
Number  fboxMissedCalls     "Missed Calls [%s]"      	{fritzboxtr064="missedCallsInDays:5"}

```




### openhab.cfg
Add the following to your openhab.cfg and configure the parameters. Or if using openHab v2 create a config file in $openhabhome/conf/services/fritzboxtr064.conf and insert the following:


```
############################# Fritz!Box TR064 Binding #######################################
#
## Binding for accessing FritzBoxes using the TR064 protocol. Uses http(s) requests.

# URL. Either use http://<fbox-ip>:49000 or https://<fbox-ip>:49443 (https preferred!)
fritzboxtr064:url=https://192.168.178.1:49443

# Refresh Interval (60000ms default)
fritzboxtr064:refresh=60000

# User Name (only use this value if you configured a user in fbox webui/config!)
# If this parameter is missing, "dslf-config" is used as default username
# It is recommended to to switch to authentication by username in fritzbox config
# and add a separate config user for this binding.
#fritzboxtr064:user=dslf-config

# PW
fritzboxtr064:pass=Fr!tZP@ssw0rd
```

## Known issues
see issues [here] (https://github.com/gitbock/fritzboxtr064/issues?q=is%3Aissue+is%3Aclosed)
 

## Debug Logging
Insert the following line into you logback.xml or logback_debug.xml inside the configuration tag.

```
<configuration scan="true">
    [...]

    <!-- FritzBox TR064 binding -->
    <logger name="org.openhab.binding.fritzboxtr064" level="DEBUG" />

    [...]
</configuration>
```
After that, watch your openhab.log file for extended log output. You may use `level="TRACE"` for even more debug information.


## Hints

### Sitemap
For the "Call" items use "Text" in your sitemap 

### Map for Presence Detection
Use a map for presence detection item:
Create file $openhab_home/configurations/transform/presence.map and add
```
-=unknown
1=present
0=not present
```
Now, as item configuration use:
```
Contact cFboxMacOnline		"Presence (Wifi) [MAP(presence.map):%d]"	<present>		{fritzboxtr064="maconline:11-22-33-44-55-66 }
```
### rule examples
If you need the caller name (resolved from the fritzbox phonebook) in a rule, extract it like this:
```
rule "Phone is ringing"
    when
        // fboxRinging is a switch item which switches to ON if call is detected
        Item fboxRinging changed from OFF to ON 
    then
            logInfo("Anrufermeldung", "Generating caller name message...")
            // fboxIncoming call receives numbers/name of incoming call
            val CallType incCall = fboxIncomingCall.state as CallType
            var callerName = incCall.destNum //destNum is external number OR resolved Name if no phonebook entry exists

            // do something with callerName

end
```

