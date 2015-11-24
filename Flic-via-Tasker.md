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
Install the Flic app and connect and name your Flic devices.

###Tasker###
Install the Tasker app on your android device.

You can [import](https://www.youtube.com/watch?v=5CCUapaRF3U)this Tasker project via an xml file ([download XML](https://drive.google.com/file/d/0B88Qoo5yy7A7T2lFa1Z3TUhtZk0/view?usp=sharing)).

Or you can create the profiles and Tasks manually.

I am using the global variables %OHSERVER and OHPORT to specify my openHAB server.
This way I can easily change the openHAB IP or port for all my tasks at once.





btw: There might be a linux library in the works, so you can connect the buttons directly to your openHAB server via a bluetooth dongle.
http://www.hardill.me.uk/wordpress/2015/10/10/flic-io-button-finally-arrived/#comment-109834