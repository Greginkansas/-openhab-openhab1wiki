# Roku
Roku provides a very simple REST API that uses GETS and POSTS. The full [External Control Guide](https://sdkdocs.roku.com/display/sdkdoc/External+Control+Guide) documents the full set of commands and queries one can make to a Roku device including sending key presses, launching channels, querying for the list of installed channels, etc. Unfortunately the one major missing piece is the ability to query for the currently active channel.

Because the API is a simple REST, the HTTP Binding or HTTP actions can be used and there is no need for a Roku specific binding.

However, unless your home router is configured to give your Rokus a static IP your Rokus will use DHCP and therefore are not guaranteed to always have the same IP address. To work around this the Rokus support Simple Service Discovery Protocol (SSDP) which lets one query the network for SSDP devices and get the URL of those devices back. Therefore it can be prudent for openHAB to discover the IP addresses of the Rokus rather than hard code them.

The following Python script will perform an SSDP query and print the Roku serial number and API URL. The script can be called from openHAB to get the Roku's current URL and then that discovered URL can be used to issue commands.

```python
#!/usr/bin/python

import sys
import socket
import re

ssdpRequest = "M-SEARCH * HTTP/1.1\r\n" + \
        "HOST: 239.255.255.250:1900\r\n" + \
        "Man: \"ssdp:discover\"\r\n" + \
        "MX: 5\r\n" + \
        "ST: roku:ecp\r\n\r\n";
socket.setdefaulttimeout(10)
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM, socket.IPPROTO_UDP)
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_TTL, 2)
sock.sendto(ssdpRequest, ("239.255.255.250", 1900))
while True:
    try:
        resp = sock.recv(1024)
        #print(resp)
        #print("Matches")
        matchObj = re.match(r'.*USN: uuid:roku:ecp:([\w\d]{12}).*LOCATION: (http://.*/).*', resp, re.S)
        if matchObj is not None:
            print (matchObj.group(1) + " " + matchObj.group(2))
    except socket.timeout:
        break
```

An example of how to use it.

## Items

```java
String BedroomRokuAddress "Bedroom Roku [%s]"
String DenRokuAddress "Den Roku [%s]"
Switch RefreshRokuAddresses // used for testing

Switch S_C_BedroomRokuHome "Go Home on Bedroom Roku"
```

## Rules

```java
rule "Get Roku Addresses"
when
        Time cron "0 0 0/1 ? * *" or
        Item RefreshRokuAddresses received command
then
    logInfo("Roku", "Refreshing Roku Addresses")
    // Replace the path with your path to the Python script
    val String results = executeCommandLine("/etc/openhab/configurations/scripts/searchRokus.py", 20000)
    logInfo("Roku", "searchRoku.py results:\n" + results)
    val rokuAddrs = results.split("\n")
    rokuAddrs.forEach[addr |
        val s = addr.split(" ")
        switch (s.get(0)) {
                // Replace 1RE######### with your Roku's serial number
                case "1RE#########" : BedroomRokuAddress.postUpdate(s.get(1))
                case "1RE#########" : DenRokuAddress.postUpdate(s.get(1))
                default : logInfo("Roku", s.get(0) + " is an unknown Roku")
        }
    ]
end

rule "Go Home on Bedroom Roku after bed"
when
        Time cron "0 0 2 ? * *" // 2 am
then
        logInfo("Roku", "Sending Bedroom Roku to Home")
        sendHttpGetRequest(BedroomRokuAddress+"keypress/Home")
end
```

NOTE: You can find the serial number on the label on the bottom of your Roku.