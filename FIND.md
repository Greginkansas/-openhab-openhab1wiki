# FIND
**THE FRAMEWORK FOR INTERNAL NAVIGATION AND DISCOVERY**

[Github : FIND](https://github.com/schollz/find)

[Main Page : FIND](https://www.internalpositioning.com/)

[MQTT Documentation : FIND](https://doc.internalpositioning.com/mqtt/)

***
Retrieve FIND client for Android through Play Store (search for "find hypercube")

Retrieve FIND server from [Github](https://github.com/schollz/find/releases).
For a Raspberry Pi, select the _arm_ binaries.
If your Pi has WiFi and you want to use it as a client for learning, too, then also retrieve and install the `fingerprint `client from this page.

Some notes on using a Pi as a client:<br>
Note it usually does not make much sense because usually you actually don't _move_ a Pi that's connected to mains power,
so you can just learn a single 'location' (one room). To use a phone is easier.
If you still want to, always remember to use `sudo`.<br>
If you encounter problems such as
`14:27:35.615 scanWifi - INFO 008 Gathering fingerprint with '/sbin/iw dev wlan0 scan -u'`
`14:27:35.625 main - WARN 009 Scan failed, will continue after a rest`
then try  `sudo ./fingerprint -iwlist`.

***

""NOTE"" this is a start to help with integration please feel free to edit and add more information.

###MQTT binding
How to use [MQTT Binding](https://github.com/openhab/openhab/wiki/MQTT-Binding) to get the data from FIND's mqtt server to OpenHAB


openhab.config file
```java
mqtt:find.url=tcp://ml.internalpositioning.com:1883
mqtt:find.clientId=OpenHAB
mqtt:find.user=YOURGROUP
mqtt:find.pwd=YOURPASSWORD
```

###Running your own FIND server
You can run your own find server like this:<br>
`pi@tvpi:~/find $ ./find-2.1-linux-arm -mosquitto findPID -mqtt mymqttserver:1883/ -mqttadmin finduser -mqttadminpass findpasswd tvpi:8003`

then it'll connect to a MQTT server on `mymqttserver` port `1883` with a 'findPID' being the process ID of the mosquitto server process, using credentials 'finduser' and password 'findpasswd'.

For usage in scripts, replace findPID above with `"`pgrep mosquitto"``.

`` ``
```` ````
``` ```

If you're using mosquitto (probably everybody does, no?), you can add authentication using `curl -X PUT "http://YOURGROUP/mqtt?group=YOURGROUP` and then FIND will automatically manage the passwd file and restart mosquitto using a SIGHUP signal. Also, use `mosquitto_passwd <passwordfile> finduser` to add credentials. `mosquitto_passwd` ain't included in the Raspian mosquitto distribution, you need to g**gle where to find it.
Find publishes to mqtt channel 'YOURGROUP/location/YOURUSER'.
You can watch mqtt events like this: `mosquitto_sub -v -h mymqttserver -p 1883 -t 'YOURGROUP/#'`

Reflect the server in openhab.cfg:
```java
mqtt:find.url=tcp://localhost:1883
mqtt:find.clientId=OpenHAB
mqtt:find.user=YOUR-OPENHAB-MOSQUITTO-USER
mqtt:find.pwd=YOUR-OPENHAB-MOSQUITTO-PASSWORD
```

###openHAB items
You need to add the username of the person to track to get the information in.
the JSONPATH will pull the current location.


location.items file
```java
String	mqqtfind_david				"David FIND [%s]"	(All)	{mqtt="<[find:YOURGROUP/location/USERNAME:state:JSONPATH($.location)]"}
```

