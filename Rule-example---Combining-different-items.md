# Excerpt
When I started playing around with OpenHAB, I first created all items and arranged them room by room in groups.
This was done quite fast and I could see everythinng I configured.
Next steps where to modify the sitemap, to arrange most important information in different frames on one initial page. This initial page was growing quickly and got way too long and confusing. I had to scroll a lot to find the block of interest. The idea was born to consolidate important information in single lines.

In the following code you can find my solution of combining heating thermostat information.
Insteat of having four different groups for actual temperature, setpoint teamperature, battery status and mode (each of the group showing 12 lines, one for each room) I now have one single block "Temperature Information" showing all information of interest realted to heating thermostats.

![](https://03752005010899291044.googlegroups.com/attach/cf940193b82ef0d1/2015-04-07%2020_13_53-Hauptmen%C3%BC.png?part=0.1&view=1&vt=ANaJVrHe2gZJSxKIiTccx8QstJfqZZhYovd2FptgxiPHcSGxPbpofxnpNLDqExAjn99D2e3PGlLoMkxQBMJqjNBhbXHT1n3J-Ys_by_ZsdR81GAPYqPXiGU)


# Description
As source code can be confusing, here are some explanations of my thoughts.
Most of the code is just stolen... borrowed from different groups, forums, or wikis. One of the main source for my ideas was the wiki page about [Reusable-Rules-via-Functions](https://github.com/openhab/openhab/wiki/Reusable-Rules-via-Functions).

I have created three functions. Actually I am not familiar with those brand new Java 8 Lambda things, but I got it somehow working. 

### function: tempLogic
This function is called from the different rules to update a map with latest information available.

### function: tempOutput
This function i actually doing the output. It just grabbs the information from the map, filled by function tempLogic.

### function: createRooms
This function is creating the map, used by the other functions.
It is called from the sysinit_rule to initiate a clean status when system starts (or rule files are reloaded)

### rule: sysinit
Is triggered when rule file is loaded. It happens on system startup and when rule file content changes and openhab detectes this change.

### rule: GF_WZ_WT_updated
The name might not be self explaining.
* GF = Ground Floor
* WZ = Livin room (Wohnzimmer in german)
* WT = Wall Thermostat   HT = Heating Thermostat

The rule is fired, when any of the readings (different Items, belonging to the same Thermostat) changes.
It than triggers the function tempLogic to update the value and afterwards ther function tempOutput to actually displaying the values.

# And this is where the magic happens (temperature.rule)

```xtend
	import org.joda.time.*
	import org.openhab.core.items.*
	import org.openhab.core.items.GenericItem
	import org.openhab.core.library.types.*
	import org.openhab.core.library.types.PercentType
	import org.openhab.core.library.items.SwitchItem
	import org.openhab.model.script.actions.*
	import org.openhab.model.script.actions.Timer
	import java.util.HashMap
	import java.util.LinkedHashMap
	import java.util.ArrayList
	import java.util.Map
	import java.util.concurrent.locks.Lock
	import java.util.concurrent.locks.ReentrantLock
	import java.util.Date
	import java.text.SimpleDateFormat




	var Map<String, org.openhab.core.items.GenericItem> outputItems = newHashMap()
	var Map<String, Map<String, Map<String, Object>>> fixtures = newHashMap()


	val org.eclipse.xtext.xbase.lib.Functions$Function3 tempLogic = [
		String room,
		String value,
		Map<String, Map<String, Map<String, Object>>> fixtures |
		
		var Map<String, Map<String, Object>> roomObj = fixtures.get(room)
		if (roomObj != null) {
			var Map<String, Object> valueObj = roomObj.get(value)
			if (valueObj != null) {
				var org.openhab.core.items.GenericItem inItem = valueObj.get("Item")
				
				if (inItem != null) {
					var doUpdate = false
					doUpdate = doUpdate || value == "Actual" && inItem.state > 0.0
					doUpdate = doUpdate || value == "Plan" &&  inItem.state > 0.0
					doUpdate = doUpdate || value == "Valve" &&  inItem.state > 0
					doUpdate = doUpdate || value == "Mode"
					doUpdate = doUpdate || value == "Battery"
					doUpdate = doUpdate && inItem.state != Uninitialized
					
					if (doUpdate) { 
						fixtures.get(room).get(value).put("LastValue", inItem.state)
						fixtures.get(room).get(value).put("LastUpdate", DateTimeUtils::currentTimeMillis())
						logDebug("Heating", String::format("Value updated! %s --> %s --> %s um %s", room, value, valueObj.get("LastValue"), valueObj.get("LastUpdate"))
					} else {
						logDebug("Heating", String::format("Value updated! %s --> %s --> no update", room, value)
					}
				}
			} else {
				if (!roomObj.containsKey(value))
					logDebug("Heating", String::format("Value not found in room %s: %s", room, value)
			}
		} else {
				logDebug("Heating", String::format("Room not found: %s", room)
		}
	]

	val org.eclipse.xtext.xbase.lib.Functions$Function3 tempOutput = [
		String room,
		Map<String, Map<String, Map<String, Object>>> fixtures,
		Map<String, org.openhab.core.items.GenericItem> outputItems	|
		
		var org.openhab.core.items.GenericItem outItem = outputItems.get(room)
		var String actual = String::format("%.1f °C", (fixtures.get(room).get("Actual").get("LastValue") as DecimalType).floatValue())
		if (actual == "0.0 °C") {
			actual = "?"
		}
		
		var String time = ""
		if ((fixtures.get(room).get("Valve") != null) && (fixtures.get(room).get("Actual").get("LastUpdate") != null) && ((fixtures.get(room).get("Actual").get("LastUpdate") as Long) > 0L)) {
			var format = new SimpleDateFormat("H:mm")
			var date = new java.util.Date()
			date.setTime(fixtures.get(room).get("Actual").get("LastUpdate"))
			time = String::format("(%s)", format.format(date))
		}
		
		var String plan = String::format("%.1f °C", (fixtures.get(room).get("Plan").get("LastValue") as DecimalType).floatValue())
		var String mode = "-"
		switch(fixtures.get(room).get("Mode").get("LastValue")) {
			case "AUTOMATIC":	mode = "A"
			case "VACATION":	mode = "U"
			case "MANUAL":		mode = "M"
			default: mode = fixtures.get(room).get("Mode").get("LastValue")
		}
		var String battery = fixtures.get(room).get("Battery").get("LastValue")
		
		logInfo("Heating", actual)
		
		if (mode == "BOOST") {
			outItem.postUpdate(String::format("%s %s", actual, mode))
		} else {
			outItem.postUpdate(String::format("%s (%s %s %s, %s)", actual, time, mode, plan, battery))
		}
	]

	val org.eclipse.xtext.xbase.lib.Functions$Function4 createRoom = [
		String room,
		java.util.ArrayList<org.openhab.core.items.GenericItem> itemList,
		Map<String, Map<String, Map<String, Object>>> fixtures,
		Map<String, org.openhab.core.items.GenericItem> outputItems |
		
		if (!fixtures.containsKey(room)) {
			var HashMap<String, Object> planMap = 	newHashMap (
									"LastUpdate" -> 0L as Long,
									"LastValue" -> new DecimalType(0.0),
									"Item" -> itemList.get(0)
								)
			
			var HashMap<String, Object> valveMap = null
			if (itemList.get(1) != null) {
				valveMap 	= 	newHashMap(
									"LastUpdate" -> 0L as Long,
									"LastValue" -> "-" as String,
									"Item" -> itemList.get(1)
								)
			}
		
			var HashMap<String, Object> batteryMap 	= 	newHashMap(
									"LastUpdate" -> 0L as Long,
									"LastValue" -> "-" as String,
									"Item" -> itemList.get(2)
								)
			
			var HashMap<String, Object> modeMap 	=	newHashMap (
									"LastUpdate" -> 0L as Long,
									"LastValue" -> "-" as String,
									"Item" -> itemList.get(3)
								 )
		
			var HashMap<String, Object> actualMap 	= 	newHashMap(
									"LastUpdate" -> 0L as Long,
									"LastValue" -> new DecimalType(0.0),
									"Item" -> itemList.get(4)
								)
			
			fixtures.put(room, newHashMap(
									"Plan" 		-> planMap,	
									"Valve" 	-> valveMap,
									"Mode" 		-> modeMap,
									"Battery" 	-> batteryMap,
									"Actual" 	-> actualMap
								) as HashMap<String, Map>
							)
		}
		
		if (!outputItems.containsKey(room)) {
			outputItems.put(room, itemList.get(5))
		}
	]


	rule sysinit
	  when
		System started
	  then
		fixtures = newHashMap() as Map<String, Map<String, Map<String, Object>>>
		outputItems = newHashMap() as Map<String, org.openhab.core.items.GenericItem>
		
		createRoom.apply("GF_WZ_WT", newArrayList(GF_WZ_WT_default, null, GF_WZ_WT_battery, GF_WZ_WT_mode, GF_WZ_WT_actual, GF_WZ_WT_komplett), fixtures, outputItems)
		createRoom.apply("GF_WZ_HT_Fenster", newArrayList(GF_WZ_HT_Fenster_default, GF_WZ_HT_Fenster_valve, GF_WZ_HT_Fenster_battery, GF_WZ_HT_Fenster_mode, GF_WZ_HT_Fenster_actual, GF_WZ_HT_Fenster_komplett), fixtures, outputItems)
		createRoom.apply("GF_WZ_HT_Wand", newArrayList(GF_WZ_HT_Wand_default, GF_WZ_HT_Wand_valve, GF_WZ_HT_Wand_battery, GF_WZ_HT_Wand_mode, GF_WZ_HT_Wand_actual, GF_WZ_HT_Wand_komplett), fixtures, outputItems)
		//createRoom.apply("GF_WC_WT", newArrayList(GF_WC_WT_default, null, GF_WC_WT_battery, GF_WC_WT_mode, GF_WC_WT_actual, GF_WC_WT_komplett), fixtures, outputItems)
		createRoom.apply("GF_WC_HT", newArrayList(GF_WC_HT_default, GF_WC_HT_valve, GF_WC_HT_battery, GF_WC_HT_mode, GF_WC_HT_actual, GF_WC_HT_komplett), fixtures, outputItems)
		//createRoom.apply("FF_WC_WT", newArrayList(FF_WC_WT_default, null, FF_WC_WT_battery, FF_WC_WT_mode, FF_WC_WT_actual, FF_WC_WT_komplett), fixtures, outputItems)
		createRoom.apply("FF_WC_HT", newArrayList(FF_WC_HT_default, FF_WC_HT_valve, FF_WC_HT_battery, FF_WC_HT_mode, FF_WC_HT_actual, FF_WC_HT_komplett), fixtures, outputItems)
		createRoom.apply("FF_BZ_WT", newArrayList(FF_BZ_WT_default, null, FF_BZ_WT_battery, FF_BZ_WT_mode, FF_BZ_WT_actual, FF_BZ_WT_komplett), fixtures, outputItems)
		createRoom.apply("FF_BZ_HT_Heizung", newArrayList(FF_BZ_HT_Heizung_default, FF_BZ_HT_Heizung_valve, FF_BZ_HT_Heizung_battery, FF_BZ_HT_Heizung_mode, FF_BZ_HT_Heizung_actual, FF_BZ_HT_Heizung_komplett), fixtures, outputItems)
		createRoom.apply("FF_BZ_HT_Handtuchheizung", newArrayList(FF_BZ_HT_Handtuchheizung_default, FF_BZ_HT_Handtuchheizung_valve, FF_BZ_HT_Handtuchheizung_battery, FF_BZ_HT_Handtuchheizung_mode, FF_BZ_HT_Handtuchheizung_actual, FF_BZ_HT_Handtuchheizung_komplett), fixtures, outputItems)
		createRoom.apply("FF_SZ_WT", newArrayList(FF_SZ_WT_default, null, FF_SZ_WT_battery, FF_SZ_WT_mode, FF_SZ_WT_actual, FF_SZ_WT_komplett), fixtures, outputItems)
		createRoom.apply("FF_SZ_HT", newArrayList(FF_SZ_HT_default, FF_SZ_HT_valve, FF_SZ_HT_battery, FF_SZ_HT_mode, FF_SZ_HT_actual, FF_SZ_HT_komplett), fixtures, outputItems)
		createRoom.apply("FF_WK_WT", newArrayList(FF_WK_WT_default, null, FF_WK_WT_battery, FF_WK_WT_mode, FF_WK_WT_actual, FF_WK_WT_komplett), fixtures, outputItems)
		createRoom.apply("FF_WK_HT_Fenster", newArrayList(FF_WK_HT_Fenster_default, FF_WK_HT_Fenster_valve, FF_WK_HT_Fenster_battery, FF_WK_HT_Fenster_mode, FF_WK_HT_Fenster_actual, FF_WK_HT_Fenster_komplett), fixtures, outputItems)
		createRoom.apply("FF_WK_HT_Balkon", newArrayList(FF_WK_HT_Balkon_default, FF_WK_HT_Balkon_valve, FF_WK_HT_Balkon_battery, FF_WK_HT_Balkon_mode, FF_WK_HT_Balkon_actual, FF_WK_HT_Balkon_komplett), fixtures, outputItems)
		createRoom.apply("SF_JZ_WT", newArrayList(SF_JZ_WT_default, null, SF_JZ_WT_battery, SF_JZ_WT_mode, SF_JZ_WT_actual, SF_JZ_WT_komplett), fixtures, outputItems)
		createRoom.apply("SF_JZ_HT", newArrayList(SF_JZ_HT_default, SF_JZ_HT_valve, SF_JZ_HT_battery, SF_JZ_HT_mode, SF_JZ_HT_actual, SF_JZ_HT_komplett), fixtures, outputItems)
		createRoom.apply("SF_CZ_WT", newArrayList(SF_CZ_WT_default, null, SF_CZ_WT_battery, SF_CZ_WT_mode, SF_CZ_WT_actual, SF_CZ_WT_komplett), fixtures, outputItems)
		createRoom.apply("SF_CZ_HT", newArrayList(SF_CZ_HT_default, SF_CZ_HT_valve, SF_CZ_HT_battery, SF_CZ_HT_mode, SF_CZ_HT_actual, SF_CZ_HT_komplett), fixtures, outputItems)
		createRoom.apply("SF_BZ_WT", newArrayList(SF_BZ_WT_default, null, SF_BZ_WT_battery, SF_BZ_WT_mode, SF_BZ_WT_actual, SF_BZ_WT_komplett), fixtures, outputItems)
		createRoom.apply("SF_BZ_HT", newArrayList(SF_BZ_HT_default, SF_BZ_HT_valve, SF_BZ_HT_battery, SF_BZ_HT_mode, SF_BZ_HT_actual, SF_BZ_HT_komplett), fixtures, outputItems)
		//createRoom.apply("SF_OFFICE_WT", newArrayList(SF_OFFICE_WT_default, null, SF_OFFICE_WT_battery, SF_OFFICE_WT_mode, SF_OFFICE_WT_actual, SF_OFFICE_WT_komplett), fixtures, outputItems)
		createRoom.apply("SF_OFFICE_HT", newArrayList(SF_OFFICE_HT_default, SF_OFFICE_HT_valve, SF_OFFICE_HT_battery, SF_OFFICE_HT_mode, SF_OFFICE_HT_actual, SF_OFFICE_HT_komplett), fixtures, outputItems)
		createRoom.apply("TF_GZ_WT", newArrayList(TF_GZ_WT_default, null, TF_GZ_WT_battery, TF_GZ_WT_mode, TF_GZ_WT_actual, TF_GZ_WT_komplett), fixtures, outputItems)
		createRoom.apply("TF_GZ_HT", newArrayList(TF_GZ_HT_default, TF_GZ_HT_valve, TF_GZ_HT_battery, TF_GZ_HT_mode, TF_GZ_HT_actual, TF_GZ_HT_komplett), fixtures, outputItems)

		for (room : fixtures.keySet()) {
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			 
			tempOutput.apply(room, fixtures, outputItems)
		}
	  end


	rule GF_WZ_WT_updated
		when
			Item GF_WZ_WT_default received update or
			Item GF_WZ_WT_battery received update or
			Item GF_WZ_WT_mode received update or
			Item GF_WZ_WT_actual received update
		then
			var room = "GF_WZ_WT"
			createRoom.apply(room, newArrayList(GF_WZ_WT_default, null, GF_WZ_WT_battery, GF_WZ_WT_mode, GF_WZ_WT_actual, GF_WZ_WT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule GF_WZ_HT_Fenster_updated
		when		
			Item GF_WZ_HT_Fenster_default received update or
			Item GF_WZ_HT_Fenster_valve received update or
			Item GF_WZ_HT_Fenster_battery received update or
			Item GF_WZ_HT_Fenster_mode received update or
			Item GF_WZ_HT_Fenster_actual received update
		then
			var room = "GF_WZ_HT_Fenster"
			createRoom.apply(room, newArrayList(GF_WZ_HT_Fenster_default, GF_WZ_HT_Fenster_valve, GF_WZ_HT_Fenster_battery, GF_WZ_HT_Fenster_mode, GF_WZ_HT_Fenster_actual, GF_WZ_HT_Fenster_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule GF_WZ_HT_Wand_updated
		when
			Item GF_WZ_HT_Wand_default received update or
			Item GF_WZ_HT_Wand_valve received update or
			Item GF_WZ_HT_Wand_battery received update or
			Item GF_WZ_HT_Wand_mode received update or
			Item GF_WZ_HT_Wand_actual received update
		then
			var room = "GF_WZ_HT_Wand"
			createRoom.apply(room, newArrayList(GF_WZ_HT_Wand_default, GF_WZ_HT_Wand_valve, GF_WZ_HT_Wand_battery, GF_WZ_HT_Wand_mode, GF_WZ_HT_Wand_actual, GF_WZ_HT_Wand_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule GF_WC_HT_updated
		when
			Item GF_WC_HT_default received update or
			Item GF_WC_HT_valve received update or
			Item GF_WC_HT_battery received update or
			Item GF_WC_HT_mode received update or
			Item GF_WC_HT_actual received update
		then
			var room = "GF_WC_HT"
			createRoom.apply(room, newArrayList(GF_WC_HT_default, null, GF_WC_HT_battery, GF_WC_HT_mode, GF_WC_HT_actual, GF_WC_HT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule FF_WC_HT_updated
		when
			Item FF_WC_HT_default received update or
			Item FF_WC_HT_valve received update or
			Item FF_WC_HT_battery received update or
			Item FF_WC_HT_mode received update or
			Item FF_WC_HT_actual received update
		then
			var room = "FF_WC_HT"
			createRoom.apply(room, newArrayList(FF_WC_HT_default, FF_WC_HT_valve, FF_WC_HT_battery, FF_WC_HT_mode, FF_WC_HT_actual, FF_WC_HT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule FF_BZ_WT_updated
		when
			Item FF_BZ_WT_default received update or
			Item FF_BZ_WT_battery received update or
			Item FF_BZ_WT_mode received update or
			Item FF_BZ_WT_actual received update
		then
			var room = "FF_BZ_WT"
			createRoom.apply(room, newArrayList(FF_BZ_WT_default, null, FF_BZ_WT_battery, FF_BZ_WT_mode, FF_BZ_WT_actual, FF_BZ_WT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule FF_BZ_HT_Heizung_updated
		when
			Item FF_BZ_HT_Heizung_default received update or
			Item FF_BZ_HT_Heizung_valve received update or
			Item FF_BZ_HT_Heizung_battery received update or
			Item FF_BZ_HT_Heizung_mode received update or
			Item FF_BZ_HT_Heizung_actual received update
		then
			var room = "FF_BZ_HT_Heizung"
			createRoom.apply(room, newArrayList(FF_BZ_HT_Heizung_default, FF_BZ_HT_Heizung_valve, FF_BZ_HT_Heizung_battery, FF_BZ_HT_Heizung_mode, FF_BZ_HT_Heizung_actual, FF_BZ_HT_Heizung_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule FF_BZ_HT_Handtuchheizung_updated
		when
			Item FF_BZ_HT_Handtuchheizung_default received update or
			Item FF_BZ_HT_Handtuchheizung_valve received update or
			Item FF_BZ_HT_Handtuchheizung_battery received update or
			Item FF_BZ_HT_Handtuchheizung_mode received update or
			Item FF_BZ_HT_Handtuchheizung_actual received update
		then
			var room = "FF_BZ_HT_Handtuchheizung"
			createRoom.apply(room, newArrayList(FF_BZ_HT_Handtuchheizung_default, FF_BZ_HT_Handtuchheizung_valve, FF_BZ_HT_Handtuchheizung_battery, FF_BZ_HT_Handtuchheizung_mode, FF_BZ_HT_Handtuchheizung_actual, FF_BZ_HT_Handtuchheizung_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule FF_SZ_WT_updated
		when
			Item FF_SZ_WT_default received update or
			Item FF_SZ_WT_battery received update or
			Item FF_SZ_WT_mode received update or
			Item FF_SZ_WT_actual received update
		then
			var room = "FF_SZ_WT"
			createRoom.apply(room, newArrayList(FF_SZ_WT_default, null, FF_SZ_WT_battery, FF_SZ_WT_mode, FF_SZ_WT_actual, FF_SZ_WT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule FF_SZ_HT_updated
		when
			Item FF_SZ_HT_default received update or
			Item FF_SZ_HT_valve received update or
			Item FF_SZ_HT_battery received update or
			Item FF_SZ_HT_mode received update or
			Item FF_SZ_HT_actual received update
		then
			var room = "FF_SZ_HT"
			createRoom.apply(room, newArrayList(FF_SZ_HT_default, FF_SZ_HT_valve, FF_SZ_HT_battery, FF_SZ_HT_mode, FF_SZ_HT_actual, FF_SZ_HT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule FF_WK_WT_updated
		when
			Item FF_WK_WT_default received update or
			Item FF_WK_WT_battery received update or
			Item FF_WK_WT_mode received update or
			Item FF_WK_WT_actual received update
		then
			var room = "FF_WK_WT"
			createRoom.apply(room, newArrayList(FF_WK_WT_default, null, FF_WK_WT_battery, FF_WK_WT_mode, FF_WK_WT_actual, FF_WK_WT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule FF_WK_HT_Fenster_updated
		when
			Item FF_WK_HT_Fenster_default received update or
			Item FF_WK_HT_Fenster_valve received update or
			Item FF_WK_HT_Fenster_battery received update or
			Item FF_WK_HT_Fenster_mode received update or
			Item FF_WK_HT_Fenster_actual received update
		then
			var room = "FF_WK_HT_Fenster"
			createRoom.apply(room, newArrayList(FF_WK_HT_Fenster_default, FF_WK_HT_Fenster_valve, FF_WK_HT_Fenster_battery, FF_WK_HT_Fenster_mode, FF_WK_HT_Fenster_actual, FF_WK_HT_Fenster_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule FF_WK_HT_Balkon_updated
		when
			Item FF_WK_HT_Balkon_default received update or
			Item FF_WK_HT_Balkon_valve received update or
			Item FF_WK_HT_Balkon_battery received update or
			Item FF_WK_HT_Balkon_mode received update or
			Item FF_WK_HT_Balkon_actual received update
		then
			var room = "FF_WK_HT_Balkon"
			createRoom.apply(room, newArrayList(FF_WK_HT_Balkon_default, FF_WK_HT_Balkon_valve, FF_WK_HT_Balkon_battery, FF_WK_HT_Balkon_mode, FF_WK_HT_Balkon_actual, FF_WK_HT_Balkon_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule SF_JZ_WT_updated
		when
			Item SF_JZ_WT_default received update or
			Item SF_JZ_WT_battery received update or
			Item SF_JZ_WT_mode received update or
			Item SF_JZ_WT_actual received update
		then
			var room = "SF_JZ_WT"
			createRoom.apply(room, newArrayList(SF_JZ_WT_default, null, SF_JZ_WT_battery, SF_JZ_WT_mode, SF_JZ_WT_actual, SF_JZ_WT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule SF_JZ_HT_updated
		when
			Item SF_JZ_HT_default received update or
			Item SF_JZ_HT_valve received update or
			Item SF_JZ_HT_battery received update or
			Item SF_JZ_HT_mode received update or
			Item SF_JZ_HT_actual received update
		then
			var room = "SF_JZ_HT"
			createRoom.apply(room, newArrayList(SF_JZ_HT_default, SF_JZ_HT_valve, SF_JZ_HT_battery, SF_JZ_HT_mode, SF_JZ_HT_actual, SF_JZ_HT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule SF_CZ_WT_updated
		when
			Item SF_CZ_WT_default received update or
			Item SF_CZ_WT_battery received update or
			Item SF_CZ_WT_mode received update or
			Item SF_CZ_WT_actual received update
		then
			var room = "SF_CZ_WT"
			createRoom.apply(room, newArrayList(SF_CZ_WT_default, null, SF_CZ_WT_battery, SF_CZ_WT_mode, SF_CZ_WT_actual, SF_CZ_WT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule SF_CZ_HT_updated
		when
			Item SF_CZ_HT_default received update or
			Item SF_CZ_HT_valve received update or
			Item SF_CZ_HT_battery received update or
			Item SF_CZ_HT_mode received update or
			Item SF_CZ_HT_actual received update
		then
			var room = "SF_CZ_HT"
			createRoom.apply(room, newArrayList(SF_CZ_HT_default, SF_CZ_HT_valve, SF_CZ_HT_battery, SF_CZ_HT_mode, SF_CZ_HT_actual, SF_CZ_HT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule SF_BZ_WT_updated
		when
			Item SF_BZ_WT_default received update or
			Item SF_BZ_WT_battery received update or
			Item SF_BZ_WT_mode received update or
			Item SF_BZ_WT_actual received update
		then
			var room = "SF_BZ_WT"
			createRoom.apply(room, newArrayList(SF_BZ_WT_default, null, SF_BZ_WT_battery, SF_BZ_WT_mode, SF_BZ_WT_actual, SF_BZ_WT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule SF_BZ_HT_updated
		when
			Item SF_BZ_HT_default received update or
			Item SF_BZ_HT_valve received update or
			Item SF_BZ_HT_battery received update or
			Item SF_BZ_HT_mode received update or
			Item SF_BZ_HT_actual received update
		then
			var room = "SF_BZ_HT"
			createRoom.apply(room, newArrayList(SF_BZ_HT_default, SF_BZ_HT_valve, SF_BZ_HT_battery, SF_BZ_HT_mode, SF_BZ_HT_actual, SF_BZ_HT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule SF_OFFICE_HT_updated
		when
			Item SF_OFFICE_HT_default received update or
			Item SF_OFFICE_HT_valve received update or
			Item SF_OFFICE_HT_battery received update or
			Item SF_OFFICE_HT_mode received update or
			Item SF_OFFICE_HT_actual received update
		then
			var room = "SF_OFFICE_HT"
			createRoom.apply(room, newArrayList(SF_OFFICE_HT_default, SF_OFFICE_HT_valve, SF_OFFICE_HT_battery, SF_OFFICE_HT_mode, SF_OFFICE_HT_actual, SF_OFFICE_HT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule TF_GZ_WT_updated
		when
			Item TF_GZ_WT_default received update or
			Item TF_GZ_WT_battery received update or
			Item TF_GZ_WT_mode received update or
			Item TF_GZ_WT_actual received update
		then
			var room = "TF_GZ_WT"
			createRoom.apply(room, newArrayList(TF_GZ_WT_default, null, TF_GZ_WT_battery, TF_GZ_WT_mode, TF_GZ_WT_actual, TF_GZ_WT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end

	rule TF_GZ_HT_updated
		when
			Item TF_GZ_HT_default received update or
			Item TF_GZ_HT_valve received update or
			Item TF_GZ_HT_battery received update or
			Item TF_GZ_HT_mode received update or
			Item TF_GZ_HT_actual received update
		then
			var room = "TF_GZ_HT"
			createRoom.apply(room, newArrayList(TF_GZ_HT_default, TF_GZ_HT_valve, TF_GZ_HT_battery, TF_GZ_HT_mode, TF_GZ_HT_actual, TF_GZ_HT_komplett), fixtures, outputItems)
			for (value : fixtures.get(room).keySet()) {
				tempLogic.apply(room, value, fixtures)
			}
			tempOutput.apply(room, fixtures, outputItems)
		end
```

# Just for the sake of completeness (haus.items)

```xtend
	Group All
	Group gGF 		(All)
	Group gFF 		(All)
	Group gSF 		(All)
	Group gTF 		(All)

	Group gGF_WZ		"Wohnzimmer"			<video> 		(gGF_WZ)
	Group gGF_WC		"Toilette"				<toilette> 		(gGF_WZ)

	Group gFF_BZ		"Badezimmer"			<bath> 			(gFF)
	Group gFF_SZ		"Schlafzimmer"			<bedroom> 		(gFF)
	Group gFF_WC		"Toilette"				<toilette> 		(gFF)
	Group gFF_WK		"Waschküche"			<bath> 			(gFF)

	Group gSF_BZ		"Badezimmer"			<bath> 			(gSF)
	Group gSF_JZ		"Jans Zimmer"			<boy2> 			(gSF)
	Group gSF_CZ		"Christinas Zimmer"		<child2> 		(gSF)
	Group gSF_OFFICE	"Büro"					<office> 		(gSF)

	Group gTF_GZ		"Gästezimmer"			<bedroom> 		(gTF)


	String ANY_DOOR		"Alle Türen [%s]"		<door>  	(All) 	//Calculated via temperaturen.rule
	String ANY_WINDOW	"Alle Fenster [%s]" 	<contact>	(All) 	//Calculated via temperaturen.rule
	String ANY_BATTERY	"Alle Fenster [%s]"		<battery>	(All) 	//Calculated via temperaturen.rule


	Switch GF_WZ_LAMPE_ECKEN "Wohnzimmer Ecken"  (gGF_WZ) {fs20="1BE4F0"}
	Switch GF_WZ_LAMPE_MITTEN "Wohnzimmer Mitten" (gGF_WZ) {fs20="1BE4F1"}
	/*String SENSOR "Funk Sensor" <gGF_WZ> {fs20="K31688069FE"}*/


	Switch GF_WZ_LAMPE_01 "Wegbeleuchtung2" (gGF_WZ) {fs20="1BE411"}
	Switch GF_WZ_LAMPE_02 "Wegbeleuchtung2" (gGF_WZ) {fs20="1BE412"}
	Switch GF_WZ_LAMPE_03 "Wegbeleuchtung2" (gGF_WZ) {fs20="1BE413"}
	Switch GF_WZ_LAMPE_04 "Wegbeleuchtung2" (gGF_WZ) {fs20="1BE414"}
	Switch GF_WZ_LAMPE_05 "Wegbeleuchtung2" (gGF_WZ) {fs20="1BE415"}
	Switch GF_WZ_LAMPE_06 "Wegbeleuchtung2" (gGF_WZ) {fs20="1BE416"}
	Switch GF_WZ_LAMPE_07 "Wegbeleuchtung2" (gGF_WZ) {fs20="1BE417"}
	Switch GF_WZ_LAMPE_08 "Wegbeleuchtung2" (gGF_WZ) {fs20="1BE418"}
	Switch GF_WZ_LAMPE_09 "Wegbeleuchtung2" (gGF_WZ) {fs20="1BE419"}
	Switch GF_WZ_LAMPE_10 "Wegbeleuchtung2" (gGF_WZ) {fs20="1BE41A"}
	Switch GF_WZ_LAMPE_11 "Wegbeleuchtung2" (gGF_WZ) {fs20="1BE41B"}
	Switch GF_WZ_LAMPE_12 "Wegbeleuchtung2" (gGF_WZ) {fs20="1BE41C"}


	/*************************************************************************************
	*									Wohnzimmer 										 *
	*************************************************************************************/

	Group 	gGF_WZ_FK_Terrasse			"Fensterkontakt Terrassentür"		<heating>		(gGF_WZ)
	Contact GF_WZ_FK_Terrasse_default	"Status [MAP(de.map):%s]" 							(gGF_WZ_FK_Terrasse) 							{ maxcube="KEQ0188673" }
	String 	GF_WZ_FK_Terrasse_battery 	"Batteriestatus [%s]" 				<battery> 		(gGF_WZ_FK_Terrasse) 							{ maxcube="KEQ0188673:type=battery" }
						
	Group 	gGF_WZ_FK_Flur				"Fensterkontakt Flurtür"			<door>			(gGF_WZ)					
	Contact GF_WZ_FK_Flur_default		"Status [MAP(de.map):%s]" 			<door>			(gGF_WZ_FK_Flur, Lights) 						{ maxcube="KEQ0187190" }
	String 	GF_WZ_FK_Flur_battery 		"Batteriestatus [%s]" 				<battery> 		(gGF_WZ_FK_Flur) 								{ maxcube="KEQ0187190:type=battery" }
						
	Group 	gGF_WZ_FK_Links				"Fensterkontakt Straße links"		<contact>		(gGF_WZ)					
	Contact GF_WZ_FK_Links_default		"Status [MAP(de.map):%s]" 			<contact>		(gGF_WZ_FK_Links, Lights) 						{ maxcube="KEQ0185936" }
	String 	GF_WZ_FK_Links_battery 		"Batteriestatus [%s]" 				<battery> 		(gGF_WZ_FK_Links) 								{ maxcube="KEQ0185936:type=battery" }
						
	Group 	gGF_WZ_FK_Kueche			"Fensterkontakt Küche"				<contact>		(gGF_WZ)					
	Contact GF_WZ_FK_Kueche_default		"Status [MAP(de.map):%s]" 			<contact>		(gGF_WZ_FK_Kueche, Lights) 						{ maxcube="LEQ0794416" }
	String 	GF_WZ_FK_Kueche_battery 	"Batteriestatus [%s]" 				<battery> 		(gGF_WZ_FK_Kueche) 								{ maxcube="LEQ0794416:type=battery" }
						
						
	Group 	gGF_WZ_WT					"Wandthermostat"					<heating>		(gGF_WZ)					
	Number 	GF_WZ_WT_default 			"SOLL-Temperatur [%.1f °C]"			<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156" }
	String 	GF_WZ_WT_battery 			"Batteriestatus [%s]" 				<battery> 		(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=battery" }
	String 	GF_WZ_WT_mode 				"Betriebsmodus [%s]" 				<calendar2> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=mode" }
	Number 	GF_WZ_WT_actual 			"IST-Temperatur [%.1f °C]" 			<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=actual" }
	Number 	GF_WZ_WT_tempComfort 		"tempcomfort [%s]" 					<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=tempcomfort" }
	Number 	GF_WZ_WT_tempEco 			"tempeco [%s]" 						<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=tempeco" }
	Number 	GF_WZ_WT_tempSetpointMax 	"tempsetpointmax [%s]" 				<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=tempsetpointmax" }
	Number 	GF_WZ_WT_tempSetpointMin 	"tempsetpointmin [%s]" 				<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=tempsetpointmin" }
	Number 	GF_WZ_WT_tempOffset 		"tempoffset [%s]" 					<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=tempoffset" }
	Number 	GF_WZ_WT_tempOpenWindow 	"tempopenwindow [%s]" 				<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=tempopenwindow" }
	Number 	GF_WZ_WT_durationOpenWindow "durationopenwindow [%s]" 			<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=durationopenwindow" }
	Number 	GF_WZ_WT_boostDuration		"boostduration [%s]" 				<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=boostduration" }
	Number 	GF_WZ_WT_boostValve 		"boostvalve [%s]" 					<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=boostvalve" }
	Number 	GF_WZ_WT_decalcification 	"decalcification [%s]" 				<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=decalcification" }
	Number 	GF_WZ_WT_valveMaximum 		"valvemaximum [%s]" 				<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=valvemaximum" }
	Number 	GF_WZ_WT_valveOffset 		"valveoffset [%s]" 					<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=valveoffset" }
	String 	GF_WZ_WT_programData 		"programdata [%s]" 					<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156:type=programdata" }
	String 	GF_WZ_WT_komplett  			"Komplett [%s]"          			<heating>  		(gGF_WZ_WT) //Calculated via temperaturen.rule
							
	Switch 	GF_WZ_WT_Setpoint			"Temperature [%.1f °C]"				<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156" }
	Switch 	GF_WZ_WT_SetMode			"Modus [%s]"						<temperature> 	(gGF_WZ_WT) 									{ maxcube="LEQ0982156=mode" }
							
	Group 	gGF_WZ_HT_Fenster					"Heizung unterm Fenster"			<heating>		(gGF_WZ)						
	Number 	GF_WZ_HT_Fenster_default 			"SOLL-Temperatur [%.1f °C]" 		<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055" }
	Number 	GF_WZ_HT_Fenster_valve 				"Ventilstellung [%.1f %%]" 			<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=valve" }
	String 	GF_WZ_HT_Fenster_battery	 		"Batteriestatus [%s]" 				<battery> 		(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=battery" }
	String 	GF_WZ_HT_Fenster_mode 				"Betriebsmodus [%s]" 				<calendar2> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=mode" }
	Number 	GF_WZ_HT_Fenster_actual 			"IST-Temperatur [%.1f °C]" 			<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=actual" }
	Number 	GF_WZ_HT_Fenster_tempComfort 		"tempcomfort [%s]" 					<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=tempcomfort" }
	Number 	GF_WZ_HT_Fenster_tempEco 			"tempeco [%s]" 						<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=tempeco" }
	Number 	GF_WZ_HT_Fenster_tempSetpointMax 	"tempsetpointmax [%s]" 				<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=tempsetpointmax" }
	Number 	GF_WZ_HT_Fenster_tempSetpointMin 	"tempsetpointmin [%s]" 				<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=tempsetpointmin" }
	Number 	GF_WZ_HT_Fenster_tempOffset 		"tempoffset [%s]" 					<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=tempoffset" }
	Number 	GF_WZ_HT_Fenster_tempOpenWindow 	"tempopenwindow [%s]" 				<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=tempopenwindow" }
	Number 	GF_WZ_HT_Fenster_durationOpenWindow	"durationopenwindow [%s]" 			<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=durationopenwindow" }
	Number 	GF_WZ_HT_Fenster_boostDuration		"boostduration [%s]" 				<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=boostduration" }
	Number 	GF_WZ_HT_Fenster_boostValve 		"boostvalve [%s]" 					<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=boostvalve" }
	Number 	GF_WZ_HT_Fenster_decalcification 	"decalcification [%s]" 				<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=decalcification" }
	Number 	GF_WZ_HT_Fenster_valveMaximum 		"valvemaximum [%s]" 				<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=valvemaximum" }
	Number 	GF_WZ_HT_Fenster_valveOffset 		"valveoffset [%s]" 					<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=valveoffset" }
	String 	GF_WZ_HT_Fenster_programData 		"programdata [%s]" 					<temperature> 	(gGF_WZ_HT_Fenster) 							{ maxcube="KEQ0380055:type=programdata" }
	String 	GF_WZ_HT_Fenster_komplett  			"Komplett [%s]"          			<heating>  		(gGF_WZ_HT_Fenster) //Calculated via temperaturen.rule
								
							
	Group 	gGF_WZ_HT_Wand				"Heizung an der Wand"				<heating>		(gGF_WZ)						
	Number 	GF_WZ_HT_Wand_default 		"SOLL-Temperatur [%.1f °C]" 		<temperature> 	(gGF_WZ_HT_Wand) 								{ maxcube="KEQ0379587" }
	Number 	GF_WZ_HT_Wand_valve 		"Ventilstellung [%.1f %%]" 			<temperature> 	(gGF_WZ_HT_Wand) 								{ maxcube="KEQ0379587:type=valve" }
	String 	GF_WZ_HT_Wand_battery 		"Batteriestatus [%s]" 				<battery> 		(gGF_WZ_HT_Wand) 								{ maxcube="KEQ0379587:type=battery" }
	String 	GF_WZ_HT_Wand_mode 			"Betriebsmodus [%s]" 				<calendar2> 	(gGF_WZ_HT_Wand) 								{ maxcube="KEQ0379587:type=mode" }
	Number 	GF_WZ_HT_Wand_actual 		"IST-Temperatur [%.1f °C]" 			<temperature> 	(gGF_WZ_HT_Wand) 								{ maxcube="KEQ0379587:type=actual" }
	String 	GF_WZ_HT_Wand_komplett  	"Komplett [%s]"          			<heating>  		(gGF_WZ_HT_Wand) //Calculated via temperaturen.rule


	/*************************************************************************************
	*									Toilette EG										 *
	*************************************************************************************/
	Group 	gGF_WC_FK_Fenster			"Fensterkontakt Fenster"			<contact>		(gGF_WC)
	Contact GF_WC_FK_Fenster_default	"Status [MAP(de.map):%s]" 			<contact>			(gGF_WC_FK_Fenster) 							{ maxcube="KEQ0186884" }
	String 	GF_WC_FK_Fenster_battery 	"Batteriestatus [%s]" 				<battery> 		(gGF_WC_FK_Fenster) 							{ maxcube="KEQ0186884:type=battery" }
							
							
	Group 	gGF_WC_FK_Flur				"Fensterkontakt Flurtür"			<door>			(gGF_WC)						
	Contact GF_WC_FK_Flur_default		"Status [MAP(de.map):%s]" 			<door>			(gGF_WC_FK_Flur) 								{ maxcube="KEQ0187188" }
	String 	GF_WC_FK_Flur_battery 		"Batteriestatus [%s]" 				<battery> 		(gGF_WC_FK_Flur) 								{ maxcube="KEQ0187188:type=battery" }
							
	/*						
	Group 	gGF_WC_WT					"Wandthermostat"					<heating>		(gGF_WC)						
	Number 	GF_WC_WT_default 			"SOLL-Temperatur [%.1f °C]"			<temperature> 	(gGF_WC_WT) 									{ maxcube="LEQ0982156" }
	String 	GF_WC_WT_battery 			"Batteriestatus [%s]" 				<battery> 		(gGF_WC_WT) 									{ maxcube="LEQ0982156:type=battery" }
	String 	GF_WC_WT_mode 				"Betriebsmodus [%s]" 				<calendar2> 	(gGF_WC_WT) 									{ maxcube="LEQ0982156:type=mode" }
	Number 	GF_WC_WT_actual 			"IST-Temperatur [%.1f °C]" 			<temperature> 	(gGF_WC_WT) 									{ maxcube="LEQ0982156:type=actual" }
	*/						
							
	Group 	gGF_WC_HT					"Heizung unterm Fenster"			<heating>		(gGF_WC)									
	Number 	GF_WC_HT_default 			"SOLL-Temperatur [%.1f °C]" 		<temperature> 	(gGF_WC_HT) 									{ maxcube="KEQ0406962" }
	Number 	GF_WC_HT_valve 				"Ventilstellung [%.1f %%]" 			<temperature> 	(gGF_WC_HT) 									{ maxcube="KEQ0406962:type=valve" }
	String 	GF_WC_HT_battery 			"Batteriestatus [%s]" 				<battery> 		(gGF_WC_HT) 									{ maxcube="KEQ0406962:type=battery" }
	String 	GF_WC_HT_mode 				"Betriebsmodus [%s]" 				<calendar2> 	(gGF_WC_HT) 									{ maxcube="KEQ0406962:type=mode" }
	Number 	GF_WC_HT_actual 			"IST-Temperatur [%.1f °C]" 			<temperature> 	(gGF_WC_HT) 									{ maxcube="KEQ0406962:type=actual" }
	String  GF_WC_HT_komplett  			"Komplett [%s]"         			<heating>  		(gGF_WC_HT) //Calculated via temperaturen.rule						
							
	/*************************************************************************************						
	*									Toliette 1 OG									 *						
	*************************************************************************************/						
	Group 	gFF_WC_FK_Fenster			"Fensterkontakt Fenster"			<contact>		(gFF_WC)						
	Contact FF_WC_FK_Fenster_default	"Status [MAP(de.map):%s]" 			<contact>		(gFF_WC_FK_Fenster) 							{ maxcube="KEQ0190577" }
	String 	FF_WC_FK_Fenster_battery 	"Batteriestatus [%s]" 				<battery> 		(gFF_WC_FK_Fenster) 							{ maxcube="KEQ0190577:type=battery" }
							
							
	Group 	gFF_WC_FK_Flur				"Fensterkontakt Flurtür"			<door>			(gFF_WC)						
	Contact FF_WC_FK_Flur_default		"Status [MAP(de.map):%s]" 			<door>			(gFF_WC_FK_Flur) 								{ maxcube="LEQ0794409" }
	String 	FF_WC_FK_Flur_battery 		"Batteriestatus [%s]" 				<battery> 		(gFF_WC_FK_Flur) 								{ maxcube="LEQ0794409:type=battery" }
							
	/*						
	Group 	gFF_WC_WT					"Wandthermostat"					<heating>		(gFF_WC)						
	Number 	FF_WC_WT_default 			"SOLL-Temperatur [%.1f °C]"			<temperature> 	(gFF_WC_WT) 									{ maxcube="LEQ0982156" }
	String 	FF_WC_WT_battery 			"Batteriestatus [%s]" 				<battery> 		(gFF_WC_WT) 									{ maxcube="LEQ0982156:type=battery" }
	String 	FF_WC_WT_mode 				"Betriebsmodus [%s]" 				<calendar2> 	(gFF_WC_WT) 									{ maxcube="LEQ0982156:type=mode" }
	Number 	FF_WC_WT_actual 			"IST-Temperatur [%.1f °C]" 			<temperature> 	(gFF_WC_WT) 									{ maxcube="LEQ0982156:type=actual" }
	String  FF_WC_WT_komplett  			"Komplett [%s]"         			<heating>  		(gFF_WC_WT) 									//Calculated via temperaturen.rule	
	*/						
							
	Group 	gFF_WC_HT					"Heizung unterm Fenster"			<heating>		(gFF_WC)									
	Number 	FF_WC_HT_default 			"SOLL-Temperatur [%.1f °C]" 		<temperature> 	(gFF_WC_HT) 									{ maxcube="KEQ0406989" }
	Number 	FF_WC_HT_valve 				"Ventilstellung [%.1f %%]" 			<temperature> 	(gFF_WC_HT) 									{ maxcube="KEQ0406989:type=valve" }
	String 	FF_WC_HT_battery 			"Batteriestatus [%s]" 				<battery> 		(gFF_WC_HT) 									{ maxcube="KEQ0406989:type=battery" }
	String 	FF_WC_HT_mode 				"Betriebsmodus [%s]" 				<calendar2> 	(gFF_WC_HT) 									{ maxcube="KEQ0406989:type=mode" }
	Number 	FF_WC_HT_actual 			"IST-Temperatur [%.1f °C]" 			<temperature> 	(gFF_WC_HT) 									{ maxcube="KEQ0406989:type=actual" }
	String  FF_WC_HT_komplett  			"Komplett [%s]"         			<heating>  		(gFF_WC_HT) 									//Calculated via temperaturen.rule						




	/*************************************************************************************
	*									Badezimmer										 *
	*************************************************************************************/
	Group 	gFF_BZ_FK_Fenster						"Fensterkontakt Fenster"			<contact>		(gFF_BZ)
	Contact FF_BZ_FK_Fenster_default				"Status [MAP(de.map):%s]" 			<contact>		(gFF_BZ_FK_Fenster) 				{ maxcube="KEQ0188710" }
	String 	FF_BZ_FK_Fenster_battery 				"Batteriestatus [%s]" 				<battery> 		(gFF_BZ_FK_Fenster) 				{ maxcube="KEQ0188710:type=battery" }

	Group 	gFF_BZ_FK_Flur							"Fensterkontakt Flurtür"			<door>			(gFF_BZ)			
	Contact FF_BZ_FK_Flur_default					"Status [MAP(de.map):%s]" 			<door>			(gFF_BZ_FK_Flur) 					{ maxcube="KEQ0185945" }
	String 	FF_BZ_FK_Flur_battery 					"Batteriestatus [%s]" 				<battery> 		(gFF_BZ_FK_Flur) 					{ maxcube="KEQ0185945:type=battery" }

	Group 	gFF_BZ_WT								"Wandthermostat"					<heating>		(gFF_BZ)			
	Number 	FF_BZ_WT_default 						"SOLL-Temperatur [%.1f °C]"			<temperature> 	(gFF_BZ_WT) 						{ maxcube="LEQ0982368" }
	String 	FF_BZ_WT_battery 						"Batteriestatus [%s]" 				<battery> 		(gFF_BZ_WT) 						{ maxcube="LEQ0982368:type=battery" }
	String 	FF_BZ_WT_mode 							"Betriebsmodus [%s]" 				<calendar2> 	(gFF_BZ_WT) 						{ maxcube="LEQ0982368:type=mode" }
	Number 	FF_BZ_WT_actual 						"IST-Temperatur [%.1f °C]" 			<temperature> 	(gFF_BZ_WT) 						{ maxcube="LEQ0982368:type=actual" }
	String	FF_BZ_WT_komplett						"Komplett [%s]"						<heating>		(gFF_BZ_WT)							//populated by temperatur.rule

	Group 	gFF_BZ_HT_Heizung						"Heizung unterm Fenster"			<heating>		(gFF_BZ)				
	Number 	FF_BZ_HT_Heizung_default 				"SOLL-Temperatur [%.1f °C]" 		<temperature> 	(gFF_BZ_HT_Heizung) 				{ maxcube="KEQ0406975" }
	Number 	FF_BZ_HT_Heizung_valve 					"Ventilstellung [%.1f %%]" 			<temperature> 	(gFF_BZ_HT_Heizung) 				{ maxcube="KEQ0406975:type=valve" }
	String 	FF_BZ_HT_Heizung_battery 				"Batteriestatus [%s]" 				<battery> 		(gFF_BZ_HT_Heizung) 				{ maxcube="KEQ0406975:type=battery" }
	String 	FF_BZ_HT_Heizung_mode 					"Betriebsmodus [%s]" 				<calendar2> 	(gFF_BZ_HT_Heizung) 				{ maxcube="KEQ0406975:type=mode" }
	Number 	FF_BZ_HT_Heizung_actual 				"IST-Temperatur [%.1f °C]" 			<temperature> 	(gFF_BZ_HT_Heizung) 				{ maxcube="KEQ0406975:type=actual" }
	String	FF_BZ_HT_Heizung_komplett				"Komplett [%s]"						<heating>		(gFF_BZ_HT_Heizung)					//populated by temperatur.rule

	Group 	gFF_BZ_HT_Handtuchheizung				"Handtuch-Heizung"					<heating>		(gFF_BZ)			
	Number 	FF_BZ_HT_Handtuchheizung_default 		"SOLL-Temperatur [%.1f °C]" 		<temperature> 	(gFF_BZ_HT_Handtuchheizung) 		{ maxcube="KEQ0407070" }
	Number 	FF_BZ_HT_Handtuchheizung_valve 			"Ventilstellung [%.1f %%]" 			<temperature> 	(gFF_BZ_HT_Handtuchheizung) 		{ maxcube="KEQ0407070:type=valve" }
	String 	FF_BZ_HT_Handtuchheizung_battery 		"Batteriestatus [%s]" 				<battery> 		(gFF_BZ_HT_Handtuchheizung) 		{ maxcube="KEQ0407070:type=battery" }
	String 	FF_BZ_HT_Handtuchheizung_mode 			"Betriebsmodus [%s]" 				<calendar2> 	(gFF_BZ_HT_Handtuchheizung) 		{ maxcube="KEQ0407070:type=mode" }
	Number 	FF_BZ_HT_Handtuchheizung_actual 		"IST-Temperatur [%.1f °C]" 			<temperature> 	(gFF_BZ_HT_Handtuchheizung) 		{ maxcube="KEQ0407070:type=actual" }
	String	FF_BZ_HT_Handtuchheizung_komplett		"Komplett [%s]"						<heating>		(gFF_BZ_HT_Handtuchheizung)			//populated by temperatur.rule



	/*************************************************************************************
	*									Schlafzimmer									 *
	*************************************************************************************/
	Group 	gFF_SZ_FK_FensterRechts					"Fensterkontakt Straße rechts"		<contact>		(gFF_SZ)
	Contact FF_SZ_FK_FensterRechts_default			"Status [MAP(de.map):%s]" 			<contact>		(gFF_SZ_FK_FensterRechts) 			{ maxcube="KEQ0190543" }
	String 	FF_SZ_FK_FensterRechts_battery 			"Batteriestatus [%s]" 				<battery> 		(gFF_SZ_FK_FensterRechts) 			{ maxcube="KEQ0190543:type=battery" }

	Group 	gFF_SZ_FK_FensterLinks					"Fensterkontakt Straße links"		<contact>		(gFF_SZ)		
	Contact FF_SZ_FK_FensterLinks_default			"Status [MAP(de.map):%s]" 			<contact>		(gFF_SZ_FK_FensterLinks) 			{ maxcube="KEQ0187146" }
	String 	FF_SZ_FK_FensterLinks_battery 			"Batteriestatus [%s]" 				<battery> 		(gFF_SZ_FK_FensterLinks) 			{ maxcube="KEQ0187146:type=battery" }

	Group 	gFF_SZ_FK_FensterSeite					"Fensterkontakt Seite"				<contact>		(gFF_SZ)		
	Contact FF_SZ_FK_FensterSeite_default			"Status [MAP(de.map):%s]" 			<contact>		(gFF_SZ_FK_FensterSeite) 			{ maxcube="KEQ0187069" }
	String 	FF_SZ_FK_FensterSeite_battery 			"Batteriestatus [%s]" 				<battery> 		(gFF_SZ_FK_FensterSeite) 			{ maxcube="KEQ0187069:type=battery" }

	Group 	gFF_SZ_FK_Waschkueche					"Fensterkontakt Waschküche"			<door>			(gFF_SZ)		
	Contact FF_SZ_FK_Waschkueche_default			"Status [MAP(de.map):%s]" 			<door>			(gFF_SZ_FK_Waschkueche) 			{ maxcube="KEQ0187216" }
	String 	FF_SZ_FK_Waschkueche_battery 			"Batteriestatus [%s]" 				<battery> 		(gFF_SZ_FK_Waschkueche) 			{ maxcube="KEQ0187216:type=battery" }

	Group 	gFF_SZ_WT								"Wandthermostat"					<heating>		(gFF_SZ)
	Number 	FF_SZ_WT_default 						"SOLL-Temperatur [%.1f °C]"			<temperature> 	(gFF_SZ_WT) 						{ maxcube="LEQ0982676" }
	String 	FF_SZ_WT_battery 						"Batteriestatus [%s]" 				<battery> 		(gFF_SZ_WT) 						{ maxcube="LEQ0982676:type=battery" }
	String 	FF_SZ_WT_mode 							"Betriebsmodus [%s]" 				<calendar2> 	(gFF_SZ_WT) 						{ maxcube="LEQ0982676:type=mode" }
	Number 	FF_SZ_WT_actual 						"IST-Temperatur [%.1f °C]" 			<temperature> 	(gFF_SZ_WT) 						{ maxcube="LEQ0982676:type=actual" }
	String	FF_SZ_WT_komplett						"Komplett [%s]"						<heating>		(gFF_SZ_WT)							//populated by temperatur.rule
				
	Group 	gFF_SZ_HT								"Heizung unterm Fenster"			<heating>		(gFF_SZ)						
	Number 	FF_SZ_HT_default 						"SOLL-Temperatur [%.1f °C]" 		<temperature> 	(gFF_SZ_HT) 						{ maxcube="KEQ0406982" }
	Number 	FF_SZ_HT_valve 							"Ventilstellung [%.1f %%]" 			<temperature> 	(gFF_SZ_HT) 						{ maxcube="KEQ0406982:type=valve" }
	String 	FF_SZ_HT_battery 						"Batteriestatus [%s]" 				<battery> 		(gFF_SZ_HT) 						{ maxcube="KEQ0406982:type=battery" }
	String 	FF_SZ_HT_mode 							"Betriebsmodus [%s]" 				<calendar2> 	(gFF_SZ_HT) 						{ maxcube="KEQ0406982:type=mode" }
	Number 	FF_SZ_HT_actual 						"IST-Temperatur [%.1f °C]" 			<temperature> 	(gFF_SZ_HT) 						{ maxcube="KEQ0406982:type=actual" }
	String	FF_SZ_HT_komplett						"Komplett [%s]"						<heating>		(gFF_SZ_HT)							//populated by temperatur.rule



	/*************************************************************************************
	*									Waschküche										 *
	*************************************************************************************/
	Group 	gFF_WK_FK_FensterRechts					"Fensterkontakt rechts"				<contact>		(gFF_WK)
	Contact FF_WK_FK_FensterRechts_default			"Status [MAP(de.map):%s]" 			<contact>		(gFF_WK_FK_FensterRechts) 			{ maxcube="LEQ0793850" }
	String 	FF_WK_FK_FensterRechts_battery 			"Batteriestatus [%s]" 				<battery> 		(gFF_WK_FK_FensterRechts) 			{ maxcube="LEQ0793850:type=battery" }
			
	Group 	gFF_WK_FK_FensterLinks					"Fensterkontakt links"				<contact>		(gFF_WK)		
	Contact FF_WK_FK_FensterLinks_default			"Status [MAP(de.map):%s]" 			<contact>		(gFF_WK_FK_FensterLinks) 			{ maxcube="LEQ0794007" }
	String 	FF_WK_FK_FensterLinks_battery 			"Batteriestatus [%s]" 				<battery> 		(gFF_WK_FK_FensterLinks) 			{ maxcube="LEQ0794007:type=battery" }
			
	Group 	gFF_WK_FK_FensterSeite					"Fensterkontakt Balkonfenster"		<contact>		(gFF_WK)		
	Contact FF_WK_FK_FensterSeite_default			"Status [MAP(de.map):%s]" 			<contact>		(gFF_WK_FK_FensterSeite) 			{ maxcube="LEQ0794477" }
	String 	FF_WK_FK_FensterSeite_battery 			"Batteriestatus [%s]" 				<battery> 		(gFF_WK_FK_FensterSeite) 			{ maxcube="LEQ0794477:type=battery" }
			
	Group 	gFF_WK_FK_Balkon						"Fensterkontakt Balkontür"			<door>			(gFF_WK)		
	Contact FF_WK_FK_Balkon_default					"Status [MAP(de.map):%s]" 			<door>			(gFF_WK_FK_Balkon) 					{ maxcube="LEQ0793895" }
	String 	FF_WK_FK_Balkon_battery 				"Batteriestatus [%s]" 				<battery> 		(gFF_WK_FK_Balkon) 					{ maxcube="LEQ0793895:type=battery" }
			
	Group 	gFF_WK_FK_Flur							"Fensterkontakt Flurtür"			<door>			(gFF_WK)		
	Contact FF_WK_FK_Flur_default					"Status [MAP(de.map):%s]" 			<door>			(gFF_WK_FK_Flur) 					{ maxcube="LEQ0794529" }
	String 	FF_WK_FK_Flur_battery 					"Batteriestatus [%s]" 				<battery> 		(gFF_WK_FK_Flur) 					{ maxcube="LEQ0794529:type=battery" }
			
	Group 	gFF_WK_WT								"Wandthermostat"					<heating>		(gFF_WK)		
	Number 	FF_WK_WT_default 						"SOLL-Temperatur [%.1f °C]"			<temperature> 	(gFF_WK_WT) 						{ maxcube="LEQ0982685" }
	String 	FF_WK_WT_battery 						"Batteriestatus [%s]" 				<battery> 		(gFF_WK_WT) 						{ maxcube="LEQ0982685:type=battery" }
	String 	FF_WK_WT_mode 							"Betriebsmodus [%s]" 				<calendar2> 	(gFF_WK_WT) 						{ maxcube="LEQ0982685:type=mode" }
	Number 	FF_WK_WT_actual 						"IST-Temperatur [%.1f °C]" 			<temperature> 	(gFF_WK_WT) 						{ maxcube="LEQ0982685:type=actual" }
	String	FF_WK_WT_komplett						"Komplett [%s]"						<heating>		(gFF_WK_WT)							//populated by temperatur.rule
			
	Group 	gFF_WK_HT_Fenster						"Heizung unterm Fenster"			<heating>		(gFF_WK)						
	Number 	FF_WK_HT_Fenster_default 				"SOLL-Temperatur [%.1f °C]" 		<temperature> 	(gFF_WK_HT_Fenster) 				{ maxcube="LEQ1004621" }
	Number 	FF_WK_HT_Fenster_valve 					"Ventilstellung [%.1f %%]" 			<temperature> 	(gFF_WK_HT_Fenster) 				{ maxcube="LEQ1004621:type=valve" }
	String 	FF_WK_HT_Fenster_battery 				"Batteriestatus [%s]" 				<battery> 		(gFF_WK_HT_Fenster) 				{ maxcube="LEQ1004621:type=battery" }
	String 	FF_WK_HT_Fenster_mode 					"Betriebsmodus [%s]" 				<calendar2> 	(gFF_WK_HT_Fenster) 				{ maxcube="LEQ1004621:type=mode" }
	Number 	FF_WK_HT_Fenster_actual 				"IST-Temperatur [%.1f °C]" 			<temperature> 	(gFF_WK_HT_Fenster) 				{ maxcube="LEQ1004621:type=actual" }
	String	FF_WK_HT_Fenster_komplett				"Komplett [%s]"						<heating>		(gFF_WK_HT_Fenster)					//populated by temperatur.rule
				
	Group 	gFF_WK_HT_Balkon						"Heizung neben Balkontür"			<heating>		(gFF_WK)						
	Number 	FF_WK_HT_Balkon_default 				"SOLL-Temperatur [%.1f °C]" 		<temperature> 	(gFF_WK_HT_Balkon) 					{ maxcube="LEQ1004590" }
	Number 	FF_WK_HT_Balkon_valve 					"Ventilstellung [%.1f %%]" 			<temperature> 	(gFF_WK_HT_Balkon) 					{ maxcube="LEQ1004590:type=valve" }
	String 	FF_WK_HT_Balkon_battery 				"Batteriestatus [%s]" 				<battery> 		(gFF_WK_HT_Balkon) 					{ maxcube="LEQ1004590:type=battery" }
	String 	FF_WK_HT_Balkon_mode 					"Betriebsmodus [%s]" 				<calendar2> 	(gFF_WK_HT_Balkon) 					{ maxcube="LEQ1004590:type=mode" }
	Number 	FF_WK_HT_Balkon_actual 					"IST-Temperatur [%.1f °C]" 			<temperature> 	(gFF_WK_HT_Balkon) 					{ maxcube="LEQ1004590:type=actual" }
	String	FF_WK_HT_Balkon_komplett				"Komplett [%s]"						<heating>		(gFF_WK_HT_Balkon)					//populated by temperatur.rule
			
			
	/*************************************************************************************		
	*										Jans										 *		
	*************************************************************************************/		
	Group 	gSF_JZ_FK_Fenster_links					"Fensterkontakt Fenster link"		<contact>		(gSF_JZ)		
	Contact SF_JZ_FK_Fenster_links_default			"Status [MAP(de.map):%s]" 			<contact>		(gSF_JZ_FK_Fenster_links) 			{ maxcube="LEQ0794474" }
	String 	SF_JZ_FK_Fenster_links_battery 			"Batteriestatus [%s]" 				<battery> 		(gSF_JZ_FK_Fenster_links) 			{ maxcube="LEQ0794474:type=battery" }
			
	Group 	gSF_JZ_FK_Fenster						"Fensterkontakt Fenster rechts"		<contact>		(gSF_JZ)		
	Contact SF_JZ_FK_Fenster_default				"Status [MAP(de.map):%s]" 			<contact>		(gSF_JZ_FK_Fenster) 				{ maxcube="LEQ0793964" }
	String 	SF_JZ_FK_Fenster_battery 				"Batteriestatus [%s]" 				<battery> 		(gSF_JZ_FK_Fenster) 				{ maxcube="LEQ0793964:type=battery" }
				
	Group 	gSF_JZ_FK_Flur							"Fensterkontakt Flurtür"			<door>			(gSF_JZ)			
	Contact SF_JZ_FK_Flur_default					"Status [MAP(de.map):%s]" 			<door>			(gSF_JZ_FK_Flur) 					{ maxcube="LEQ0794752" }
	String 	SF_JZ_FK_Flur_battery 					"Batteriestatus [%s]" 				<battery> 		(gSF_JZ_FK_Flur) 					{ maxcube="LEQ0794752:type=battery" }
			
	Group 	gSF_JZ_WT								"Wandthermostat"					<heating>		(gSF_JZ)			
	Number 	SF_JZ_WT_default 						"SOLL-Temperatur [%.1f °C]"			<temperature> 	(gSF_JZ_WT) 						{ maxcube="LEQ0982677" }
	String 	SF_JZ_WT_battery 						"Batteriestatus [%s]" 				<battery> 		(gSF_JZ_WT) 						{ maxcube="LEQ0982677:type=battery" }
	String 	SF_JZ_WT_mode 							"Betriebsmodus [%s]" 				<calendar2> 	(gSF_JZ_WT) 						{ maxcube="LEQ0982677:type=mode" }
	Number 	SF_JZ_WT_actual 						"IST-Temperatur [%.1f °C]" 			<temperature> 	(gSF_JZ_WT) 						{ maxcube="LEQ0982677:type=actual" }
	String	SF_JZ_WT_komplett						"Komplett [%s]"						<heating>		(gSF_JZ_WT)							//populated by temperatur.rule
			
	Group 	gSF_JZ_HT								"Heizung unterm Fenster"			<heating>		(gSF_JZ)						
	Number 	SF_JZ_HT_default 						"SOLL-Temperatur [%.1f °C]" 		<temperature> 	(gSF_JZ_HT) 						{ maxcube="LEQ1004600" }
	Number 	SF_JZ_HT_valve 							"Ventilstellung [%.1f %%]" 			<temperature> 	(gSF_JZ_HT) 						{ maxcube="LEQ1004600:type=valve" }
	String 	SF_JZ_HT_battery 						"Batteriestatus [%s]" 				<battery> 		(gSF_JZ_HT) 						{ maxcube="LEQ1004600:type=battery" }
	String 	SF_JZ_HT_mode 							"Betriebsmodus [%s]" 				<calendar2> 	(gSF_JZ_HT) 						{ maxcube="LEQ1004600:type=mode" }
	Number 	SF_JZ_HT_actual 						"IST-Temperatur [%.1f °C]" 			<temperature> 	(gSF_JZ_HT) 						{ maxcube="LEQ1004600:type=actual" }
	String	SF_JZ_HT_komplett						"Komplett [%s]"						<heating>		(gSF_JZ_HT)							//populated by temperatur.rule

	//Switch FritzDECT200	"Fritz!DECT200"	 (gSF_JZ) { fritzaha="7390,087610024850" }

	/*************************************************************************************		
	*										Christinas									 *		
	*************************************************************************************/		
	Group 	gSF_CZ_FK_Fenster						"Fensterkontakt Fenster"			<contact>		(gSF_CZ)		
	Contact SF_CZ_FK_Fenster_default				"Status [MAP(de.map):%s]" 			<contact>		(gSF_CZ_FK_Fenster) 				{ maxcube="LEQ0794499" }
	String 	SF_CZ_FK_Fenster_battery 				"Batteriestatus [%s]" 				<battery> 		(gSF_CZ_FK_Fenster) 				{ maxcube="LEQ0794499:type=battery" }
							
							
	Group 	gSF_CZ_FK_Flur							"Fensterkontakt Flurtür"			<door>			(gSF_CZ)			
	Contact SF_CZ_FK_Flur_default					"Status [MAP(de.map):%s]" 			<door>			(gSF_CZ_FK_Flur) 					{ maxcube="LEQ0793406" }
	String 	SF_CZ_FK_Flur_battery 					"Batteriestatus [%s]" 				<battery> 		(gSF_CZ_FK_Flur) 					{ maxcube="LEQ0793406:type=battery" }
			
	Group 	gSF_CZ_WT								"Wandthermostat"					<heating>		(gSF_CZ)			
	Number 	SF_CZ_WT_default 						"SOLL-Temperatur [%.1f °C]"			<temperature> 	(gSF_CZ_WT) 						{ maxcube="LEQ0982671" }
	String 	SF_CZ_WT_battery 						"Batteriestatus [%s]" 				<battery> 		(gSF_CZ_WT) 						{ maxcube="LEQ0982671:type=battery" }
	String 	SF_CZ_WT_mode 							"Betriebsmodus [%s]" 				<calendar2> 	(gSF_CZ_WT) 						{ maxcube="LEQ0982671:type=mode" }
	Number 	SF_CZ_WT_actual 						"IST-Temperatur [%.1f °C]" 			<temperature> 	(gSF_CZ_WT) 						{ maxcube="LEQ0982671:type=actual" }
	String	SF_CZ_WT_komplett						"Komplett [%s]"						<heating>		(gSF_CZ_WT)							//populated by temperatur.rule
			
	Group 	gSF_CZ_HT								"Heizung unterm Fenster"			<heating>		(gSF_CZ)						
	Number 	SF_CZ_HT_default 						"SOLL-Temperatur [%.1f °C]" 		<temperature> 	(gSF_CZ_HT) 						{ maxcube="LEQ1004588" }
	Number 	SF_CZ_HT_valve 							"Ventilstellung [%.1f %%]" 			<temperature> 	(gSF_CZ_HT) 						{ maxcube="LEQ1004588:type=valve" }
	String 	SF_CZ_HT_battery 						"Batteriestatus [%s]" 				<battery> 		(gSF_CZ_HT) 						{ maxcube="LEQ1004588:type=battery" }
	String 	SF_CZ_HT_mode 							"Betriebsmodus [%s]" 				<calendar2> 	(gSF_CZ_HT) 						{ maxcube="LEQ1004588:type=mode" }
	Number 	SF_CZ_HT_actual 						"IST-Temperatur [%.1f °C]" 			<temperature> 	(gSF_CZ_HT) 						{ maxcube="LEQ1004588:type=actual" }
	String	SF_CZ_HT_komplett						"Komplett [%s]"						<heating>		(gSF_CZ_HT)							//populated by temperatur.rule
			
			
			
	/*************************************************************************************		
	*									Badezimmer 2. OG								 *		
	*************************************************************************************/		
	Group 	gSF_BZ_FK_Fenster						"Fensterkontakt Fenster"			<contact>		(gSF_BZ)		
	Contact SF_BZ_FK_Fenster_default				"Status [MAP(de.map):%s]" 			<contact>		(gSF_BZ_FK_Fenster) 				{ maxcube="LEQ0793862" }
	String 	SF_BZ_FK_Fenster_battery 				"Batteriestatus [%s]" 				<battery> 		(gSF_BZ_FK_Fenster) 				{ maxcube="LEQ0793862:type=battery" }
			
	Group 	gSF_BZ_FK_Flur							"Fensterkontakt Flurtür"			<door>			(gSF_BZ)			
	Contact SF_BZ_FK_Flur_default					"Status [MAP(de.map):%s]" 			<door>			(gSF_BZ_FK_Flur) 					{ maxcube="LEQ0794069" }
	String 	SF_BZ_FK_Flur_battery 					"Batteriestatus [%s]" 				<battery> 		(gSF_BZ_FK_Flur) 					{ maxcube="LEQ0794069:type=battery" }
			
	Group 	gSF_BZ_WT								"Wandthermostat"					<heating>		(gSF_BZ)			
	Number 	SF_BZ_WT_default 						"SOLL-Temperatur [%.1f °C]"			<temperature> 	(gSF_BZ_WT) 						{ maxcube="LEQ0982672" }
	String 	SF_BZ_WT_battery 						"Batteriestatus [%s]" 				<battery> 		(gSF_BZ_WT) 						{ maxcube="LEQ0982672:type=battery" }
	String 	SF_BZ_WT_mode 							"Betriebsmodus [%s]" 				<calendar2> 	(gSF_BZ_WT) 						{ maxcube="LEQ0982672:type=mode" }
	Number 	SF_BZ_WT_actual 						"IST-Temperatur [%.1f °C]" 			<temperature> 	(gSF_BZ_WT) 						{ maxcube="LEQ0982672:type=actual" }
	String	SF_BZ_WT_komplett						"Komplett [%s]"						<heating>		(gSF_BZ_WT)							//populated by temperatur.rule
			
	Group 	gSF_BZ_HT								"Heizung"							<heating>		(gSF_BZ)						
	Number 	SF_BZ_HT_default 						"SOLL-Temperatur [%.1f °C]" 		<temperature> 	(gSF_BZ_HT) 						{ maxcube="LEQ1004562" }
	Number 	SF_BZ_HT_valve 							"Ventilstellung [%.1f %%]" 			<temperature> 	(gSF_BZ_HT) 						{ maxcube="LEQ1004562:type=valve" }
	String 	SF_BZ_HT_battery 						"Batteriestatus [%s]" 				<battery> 		(gSF_BZ_HT) 						{ maxcube="LEQ1004562:type=battery" }
	String 	SF_BZ_HT_mode 							"Betriebsmodus [%s]" 				<calendar2> 	(gSF_BZ_HT) 						{ maxcube="LEQ1004562:type=mode" }
	Number 	SF_BZ_HT_actual 						"IST-Temperatur [%.1f °C]" 			<temperature> 	(gSF_BZ_HT) 						{ maxcube="LEQ1004562:type=actual" }
	String	SF_BZ_HT_komplett						"Komplett [%s]"						<heating>		(gSF_BZ_HT)							//populated by temperatur.rule



	/*************************************************************************************
	*										Büro										 *
	*************************************************************************************/
	Group 	gSF_OFFICE_FK_Fenster						"Fensterkontakt Fenster"			<contact>			(gSF_OFFICE)
	Contact SF_OFFICE_FK_Fenster_default				"Status [MAP(de.map):%s]" 			<contact>			(gSF_OFFICE_FK_Fenster) 			{ maxcube="LEQ0794060" }
	String 	SF_OFFICE_FK_Fenster_battery 				"Batteriestatus [%s]" 				<battery> 			(gSF_OFFICE_FK_Fenster) 			{ maxcube="LEQ0794060:type=battery" }
							
							
	Group 	gSF_OFFICE_FK_Flur							"Fensterkontakt Flurtür"			<door>				(gSF_OFFICE)		
	Contact SF_OFFICE_FK_Flur_default					"Status [MAP(de.map):%s]" 			<door>				(gSF_OFFICE_FK_Flur) 				{ maxcube="LEQ0794124" }
	String 	SF_OFFICE_FK_Flur_battery 					"Batteriestatus [%s]" 				<battery> 			(gSF_OFFICE_FK_Flur) 				{ maxcube="LEQ0794124:type=battery" }
							
	/*						
	Group 	gSF_OFFICE_WT								"Wandthermostat"					<heating>			(gSF_OFFICE)		
	Number 	SF_OFFICE_WT_default 						"SOLL-Temperatur [%.1f °C]"			<temperature> 		(gSF_OFFICE_WT) 					{ maxcube="LEQ0982156" }
	String 	SF_OFFICE_WT_battery 						"Batteriestatus [%s]" 				<battery> 			(gSF_OFFICE_WT) 					{ maxcube="LEQ0982156:type=battery" }
	String 	SF_OFFICE_WT_mode 							"Betriebsmodus [%s]" 				<calendar2> 		(gSF_OFFICE_WT) 					{ maxcube="LEQ0982156:type=mode" }
	Number 	SF_OFFICE_WT_actual 						"IST-Temperatur [%.1f °C]" 			<temperature> 		(gSF_OFFICE_WT) 					{ maxcube="LEQ0982156:type=actual" }
	String	SF_OFFICE_WT_komplett						"Komplett [%s]"						<heating>			(gSF_OFFICE_WT)						//populated by temperatur.rule
	*/									
			
	Group 	gSF_OFFICE_HT								"Heizung"							<heating>			(gSF_OFFICE)					
	Number 	SF_OFFICE_HT_default 						"SOLL-Temperatur [%.1f °C]" 		<temperature> 		(gSF_OFFICE_HT) 					{ maxcube="LEQ1004597" }
	Number 	SF_OFFICE_HT_valve 							"Ventilstellung [%.1f %%]" 			<temperature> 		(gSF_OFFICE_HT) 					{ maxcube="LEQ1004597:type=valve" }
	String 	SF_OFFICE_HT_battery 						"Batteriestatus [%s]" 				<battery> 			(gSF_OFFICE_HT) 					{ maxcube="LEQ1004597:type=battery" }
	String 	SF_OFFICE_HT_mode 							"Betriebsmodus [%s]" 				<calendar2> 		(gSF_OFFICE_HT) 					{ maxcube="LEQ1004597:type=mode" }
	Number 	SF_OFFICE_HT_actual 						"IST-Temperatur [%.1f °C]" 			<temperature> 		(gSF_OFFICE_HT) 					{ maxcube="LEQ1004597:type=actual" }
	String	SF_OFFICE_HT_komplett						"Komplett [%s]"						<heating>			(gSF_OFFICE_HT)						//populated by temperatur.rule



	/*************************************************************************************
	*										Gästezimmer									 *
	*************************************************************************************/
	Group 	gTF_GZ_FK_FensterGarteNRechts				"Fensterkontakt Fenster Garten Rechts"	<contact>		(gTF_GZ)
	Contact TF_GZ_FK_FensterGarteNRechts_default		"Status [MAP(de.map):%s]" 				<contact>		(gTF_GZ_FK_FensterGartenRechts) 	{ maxcube="LEQ0793904" }
	String 	TF_GZ_FK_FensterGarteNRechts_battery 		"Batteriestatus [%s]" 					<battery> 		(gTF_GZ_FK_FensterGartenRechts) 	{ maxcube="LEQ0793904:type=battery" }

	Group 	gTF_GZ_FK_FensterGartenLinks				"Fensterkontakt Fenster Garten links"	<contact>		(gTF_GZ)
	Contact TF_GZ_FK_FensterGartenLinks_default			"Status [MAP(de.map):%s]" 				<contact>		(gTF_GZ_FK_FensterGartenLinks) 		{ maxcube="LEQ0793853" }
	String 	TF_GZ_FK_FensterGartenLinks_battery 		"Batteriestatus [%s]" 					<battery> 		(gTF_GZ_FK_FensterGartenLinks) 		{ maxcube="LEQ0793853:type=battery" }

	Group 	gTF_GZ_FK_FensterStrasse					"Fensterkontakt Fenster Straße"			<contact>		(gTF_GZ)
	Contact TF_GZ_FK_FensterStrasse_default				"Status [MAP(de.map):%s]" 				<contact>		(gTF_GZ_FK_FensterStrasse) 			{ maxcube="LEQ0794565" }
	String 	TF_GZ_FK_FensterStrasse_battery 			"Batteriestatus [%s]" 					<battery> 		(gTF_GZ_FK_FensterStrasse) 			{ maxcube="LEQ0794565:type=battery" }

	Group 	gTF_GZ_WT									"Wandthermostat"						<heating>		(gTF_GZ)	
	Number 	TF_GZ_WT_default 							"SOLL-Temperatur [%.1f °C]"				<temperature> 	(gTF_GZ_WT) 						{ maxcube="LEQ0981729" }
	String 	TF_GZ_WT_battery 							"Batteriestatus [%s]" 					<battery> 		(gTF_GZ_WT) 						{ maxcube="LEQ0981729:type=battery" }
	String 	TF_GZ_WT_mode 								"Betriebsmodus [%s]" 					<calendar2> 	(gTF_GZ_WT) 						{ maxcube="LEQ0981729:type=mode" }
	Number 	TF_GZ_WT_actual 							"IST-Temperatur [%.1f °C]" 				<temperature> 	(gTF_GZ_WT) 						{ maxcube="LEQ0981729:type=actual" }
	String	TF_GZ_WT_komplett							"Komplett [%s]"							<heating>		(gTF_GZ_WT)							//populated by temperatur.rule

	Group 	gTF_GZ_HT									"Heizung"								<heating>		(gTF_GZ)						
	Number 	TF_GZ_HT_default 							"SOLL-Temperatur [%.1f °C]" 			<temperature> 	(gTF_GZ_HT) 						{ maxcube="LEQ1004560" }
	Number 	TF_GZ_HT_valve 								"Ventilstellung [%.1f %%]" 				<temperature> 	(gTF_GZ_HT) 						{ maxcube="LEQ1004560:type=valve" }
	String 	TF_GZ_HT_battery 							"Batteriestatus [%s]" 					<battery> 		(gTF_GZ_HT) 						{ maxcube="LEQ1004560:type=battery" }
	String 	TF_GZ_HT_mode 								"Betriebsmodus [%s]" 					<calendar2> 	(gTF_GZ_HT) 						{ maxcube="LEQ1004560:type=mode" }
	Number 	TF_GZ_HT_actual 							"IST-Temperatur [%.1f °C]" 				<temperature> 	(gTF_GZ_HT) 						{ maxcube="LEQ1004560:type=actual" }
	String	TF_GZ_HT_komplett							"Komplett [%s]"							<heating>		(gTF_GZ_HT)							//populated by temperatur.rule
```