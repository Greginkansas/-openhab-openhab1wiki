# FIND
**THE FRAMEWORK FOR INTERNAL NAVIGATION AND DISCOVERY**

[Github : FIND](https://github.com/schollz/find)

[Main Page : FIND](https://www.internalpositioning.com/)

***

""NOTE"" this is a start to help with integration please feel free to edit and add more information.

***

Here is how I use [MQTT Binding](https://github.com/openhab/openhab/wiki/MQTT-Binding) to get the data from FIND mqtt server to OpenHAB


openhab.config file
```java
mqtt:find.url=tcp://ml.internalpositioning.com:1883
mqtt:find.clientId=OpenHAB
mqtt:find.user=YOURGROUP
mqtt:find.pwd=YOURPASSWORD
```

I have found that you need to add the username of the person to track to get the information in.
the JSONPATH will pull the current location.


location.items file
```java
String	mqqtfind_david				"David FIND [%s]"	(All)	{mqtt="<[find:YOURGROUP/location/USERNAME:state:JSONPATH($.location)]"}
```

