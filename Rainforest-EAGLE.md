#Rainforest EAGLE™ Energy Access Gateway#
The Eagle lets you retrieve the data from your smart meter locally with out having to pull the data from the cloud.

You will need to add the gateway to your energy meter through your electric company. Here in Texas there is a website devoted for smart meters. [SmartMeterTexas.com](https://www.smartmetertexas.com)

After the gateway is added to the meter you will need to disable local security.

In the web portal click the down arrow next to date.
at the bottom of that page click the gear.

![Gear](https://community-openhab-org.s3-eu-central-1.amazonaws.com/original/2X/b/b54f1bca8430eb29a53184b9ad07cc708547c6e2.jpg)

uncheck the security box.

![Security](https://community-openhab-org.s3-eu-central-1.amazonaws.com/original/2X/a/aadfb7a13c7363826c90a9add8d7a99c381649ec.jpg)

The rule and code courtesy of [Mark Clark](https://community.openhab.org/users/guessed/activity)

In my setup I created separate item and rule files for power.

### power.items
    Group	GPersist (All)
    Group	GMonitorPower
    Group	GMonitorEnergy
    
    Number   HousePowerInstant    "SmartMeter Power [%.3f]" 	<energy>	(GMonitorPower,GPersist)
    Number   HouseEnergyDelivered "SmartMeter Energy Delivered [%.3f]" (GMonitorEnergy,GPersist)
    Number   HouseEnergySent      "SmartMeter Energy Sent [%.3f]" (GMonitorEnergy,GPersist)
    Number   HouseEnergyPrice     "SmartMeter Energy Price [%.3f]" (GMonitorEnergy,GPersist)
    Number   HouseEnergyCost      "SmartMeter Energy Cost [%.3f]" (GMonitorEnergy,GPersist)

### power.rules
    import org.openhab.core.library.types.*
    import java.lang.Float
    import java.lang.Long
    import org.joda.time.DateTime
    
    var DateTime lastTimestamp = null
    var float lastDelivered
    var float lastReceived
    
    rule "Pull Data from Eagle"
      when
        Time cron "0 0-59 * * * ?"
      then
              var t = now
        val String EAGLE_MAC = "0xe1e100000e1e10"
        val String EAGLE_URL = "http://192.168.1.13/cgi-bin/cgi_manager"
    
        var String postData = String::format("<LocalCommand>
      <Name>get_usage_data</Name>
      <MacId>%s</MacId>
    </LocalCommand>", EAGLE_MAC, EAGLE_MAC)
    
        var result = sendHttpPostRequest(EAGLE_URL, "application/x-www-form-urlencoded", postData)
        // logDebug("eagle", result)
           
        try {
          var long timestamp = Long::parseLong(transform("JSONPATH", "$.demand_timestamp", result))
          var DateTime currTimestamp = new DateTime(timestamp * 1000)
          var float price = Float::parseFloat(transform("JSONPATH", "$.price", result))
          var float currDemand = Float::parseFloat(transform("JSONPATH", "$.demand", result))
          var float currDelivered = Float::parseFloat(transform("JSONPATH", "$.summation_delivered", result))
          var float currReceived = Float::parseFloat(transform("JSONPATH", "$.summation_received", result))
      
          postUpdate(HousePowerInstant, currDemand * 1000)
    
          if (lastTimestamp != null && !lastTimestamp.equals(currTimestamp)) {
            var float used = currDelivered - lastDelivered
            var float sent = currReceived - lastReceived
    
            logDebug("eagle", String::format("Energy %s demand=%.3f received=%.3f delivered=%.3f", currTimestamp.toString,
              currDemand, used, sent))
            postUpdate(HouseEnergySent, sent * 1000)
            postUpdate(HouseEnergyDelivered, used * 1000)
            postUpdate(HouseEnergyCost, (used - sent) * 1000 * price)
          }
    
          postUpdate(HouseEnergyPrice, price)
    
          lastDelivered = currDelivered
          lastReceived = currReceived
          lastTimestamp = currTimestamp  
        } catch (NumberFormatException nfe) {
          logError("eagle", "Bad Data " + result.replaceAll("\n", " "))
        }
        var long x = now.getMillis - t.getMillis
        logInfo("eagle", "PERF Pull-Data-from-Eagle elapsed: " + String::valueOf(x) + "ms")
    end

