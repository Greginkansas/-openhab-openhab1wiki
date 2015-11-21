### Overview
The [SiteWhere](http://www.sitewhere.org/) persistence provider allows openHAB data to be forwarded to a SiteWhere
server instance running locally or in the cloud. All events for an openHAB instance will be stored in SiteWhere
under a virtual device with hardware id specified in the persistence provider implementation. Events are
delivered via the SiteWhere agent which uses the MQTT protocol. The SiteWhere 
[administrative application](http://documentation.sitewhere.org/userguide/adminui/adminui.html) may be 
used to view data for the virtual device. It can also be used to issue commands to items in openHAB based
on the SiteWhere command framework. See [this tutorial](http://documentation.sitewhere.org/integration/openhab.html) 
for a step-by-step walkthrough.