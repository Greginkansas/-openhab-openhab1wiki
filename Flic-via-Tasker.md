#Flic (Bluetooth Buttons) via Tasker (Android)#
You can connect your [Flic](http://flic.io) Bluetooth Buttons via [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) to openHAB.

##openHAB configuration##
###items###
flic.items
```
Group gFlic
Group gFlic01	(gFlic)

Switch		Flic01click		"Flic01 Click"	{ autoupdate="false"}
Switch		Flic01double	"Flic01 Double Click"	{ autoupdate="false"}
Switch		Flic01hold		"Flic01 Hold"	{ autoupdate="false"}
Switch		Flic01down		"Flic01 Down"
```

The items are defined as push buttons via `{autoupdate="false"}`.
You can use a Flic as a normal button by using the "up" and "down" function. (follow the Flic01down item in this guide)

###sitemap###
flic.sitemap
```
sitemap flic label="Flic"
{	
	Frame label="Flic 01" {
		Switch		item=Flic01click
		Switch		item=Flic01double
		Switch		item=Flic01hold
		Switch		item=Flic01down
	}
}
```

###rules###
flic.rules
```
rule "Flic01 Click"
	when 
		Item Flic01click received command
	then
		logInfo("Flic", "Flic01 Click")
end

rule "Flic01 Double Click"
	when 
		Item Flic01double received command
	then
		logInfo("Flic", "Flic01 Double Click")
end

rule "Flic01 Hold"
	when 
		Item Flic01hold received command
	then
		logInfo("Flic", "Flic01 Hold")
end

rule "Flic01 Down"
	when 
		Item Flic01down changed from OFF to ON
	then
		logInfo("Flic", "Flic01 Down")
end

rule "Flic01 Up"
	when 
		Item Flic01down changed from ON to OFF
	then
		logInfo("Flic", "Flic01 Up")
end
```

##configure your android device##
###Flic app###
Install the Flic app and connect your Flic devices.
Call them "Flic01" to "Flic0.." for this guide.

###Tasker###
Install the Tasker app on your android device.

You can [import](https://www.youtube.com/watch?v=5CCUapaRF3U) this Tasker project via an xml file ([download XML](https://drive.google.com/file/d/0B88Qoo5yy7A7T2lFa1Z3TUhtZk0/view?usp=sharing)).

Use the global variables %OHSERVER and OHPORT to specify the openHAB server.
This way you can easily change the openHAB IP or port for all tasks at once.

###profile###
If you have the Flic app installed there will be a tasker plugin available when you add a new profile and select "event".
Click on "Configuration" and select the FlicButton and the Type (of Action) you want to trigger on.

###tasks###
The tasks use local variables to define the number of each FlicButton (%flic_nr), the action (%flic_action) and the state (%flic_state) you want to put the item in.

You can clone the first task and just change the value of these variables to create the next task. (e.g. substitute "click" for "hold" or "01" for "02").

If the task gets executed it sends an HTTP GET request to the REST API of the openHAB server and the rule will print a line in the log to show the command was received.


##without Android/Tasker##
btw: There might be a linux library in the works, so you can connect the buttons directly to your openHAB server via a bluetooth dongle.
http://www.hardill.me.uk/wordpress/2015/10/10/flic-io-button-finally-arrived/#comment-109834