
* [LG Hombots](#LG-Hombots)
* [WiFi hack](#wifi-hack)
* [HTTP binding based control](#HTTP-binding-based-control)

## LG Hombots

LG offers a range of robot vacuum cleaners called e.g. VR6470.
Unlike the probably more popular iRobot Roomba models, the Hombot firmware is Linux based, allowing for external control and integration in openHAB.
In principle, any model will do (some hints: prices vary greatly, but the different national versions are not different, almost all features are software only, and the last two or three digits refer to the color only (really!), so get a cheap one. My one's a VR64607 bought on amazon.es). Check out the forum to host the install guide for more details.

## WiFi hack
You need to equip your Hombot with a WiFi stick first .
See [this install guide](http://www.roboter-forum.com/showthread.php?10009-LG-Hombot-3-0-%28VR6260-VR6270-VR6340%29-WLAN-Steuerung-per-Weboberfl%E4che&p=107354&viewfull=1#post107354) (registration required).

## HTTP binding based control
in items:

``String Hombot   "Luigi, der Hombot"                     <luigi>         (Status,Test)   { http=">[1:GET:http://192.168.1.2:6260/json.cgi?%%7b%%22COMMAND%%22:%%22CLEAN_START%%22%%7d] >[0:GET:http://192.168.1.2:6260/json.cgi?%%7b%%22COMMAND%%22:%%22PAUSE%%22%%7d] >[2:GET:http://192.168.1.2:6260/json.cgi?%%7b%%22COMMAND%%22:%%22HOMING%%22%%7d] <[http://192.168.1.2:6260/status.html:5000:REGEX(.*<b>Robot-state</b>: <status>(.+)</status>.*)]", autoupdate="false" }``

in sitemap:

``Switch item=Hombot label="Luigi, der Hombot [Status %s]" mappings=[1="Clean", 2="Charge", 0="Pause"]``
