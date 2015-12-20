**Note**: This Script-Engine comes with no forward compatibility guarantee. It is currently untested against openHAB2 and therefore it can happen that scripts need to be rewritten for openHAB2 compatibility.

# Script-Engines

This addon allows the usage of different scripting languages as basis for rules. It was tested with the Jython-Scripting-Runtime and all examples rely on this scripting language. Feel free to contribute to this wiki, if you are using another language like JRuby.

## Installation
The package needs to be installed to addons like any other addon [openhab]/addons. By default it comes with build-in Nashorn-Engine for JavaScript (\*.js) as scripting language. Feel free to use another scripting language like Jython (\*.py). Put all your scripts in the [openhab]/scripts directory.

- **ATTENTION**: The addon jsr223 jar of 1.7.0 is broken, please use a [1.8.0 build](https://openhab.ci.cloudbees.com/job/openHAB/)

**Requirements, using Nashorn-Engine for JavaScript**
- Use Java Version >= 1.8.0_40
- Use openHAB Version >= 1.8.0
- For loading Java classes with Nashorn you need to start openHab with the for java 8 preferred class-loader. Therefore add the ````-Dorg.osgi.framework.bundle.parent=ext```` parameter to your start.sh or start.bat.


### Jython-Installation
- Download Jython 2.7.0 from http://www.jython.org/downloads.html as an installer package.
- Install jython to /opt/jython
- customize start.sh to reference to the JYTHON-PATH:
    
```
#!/bin/sh

cd `dirname $0`

# set path to eclipse folder. If local folder, use '.'; otherwise, use /path/to/eclipse/
eclipsehome="server";

# set ports for HTTP(S) server
HTTP_PORT=8080
HTTPS_PORT=8443

JYTHON_HOME="/opt/jython";

# get path to equinox jar inside $eclipsehome folder
cp=$(echo lib/*.jar | tr ' ' ':'):$(find $eclipsehome -name "org.eclipse.equinox.launcher_*.jar" | sort | tail -1);
echo $cp
echo Launching the openHAB runtime...
java \
    -Dpython.home="$JYTHON_HOME" \
    -Dosgi.clean=true \
    -Declipse.ignoreApp=true \
    -Dosgi.noShutdown=true  \
    -Djetty.port=$HTTP_PORT  \
    -Djetty.port.ssl=$HTTPS_PORT \
    -Djetty.home=.  \
    -Dlogback.configurationFile=configurations/logback.xml \
    -Dfelix.fileinstall.dir=addons -Dfelix.fileinstall.filter=.*\\.jar \
    -Djava.library.path=lib \
    -Djava.security.auth.login.config=./etc/login.conf \
    -Dorg.quartz.properties=./etc/quartz.properties \
    -Dequinox.ds.block_timeout=240000 \
    -Dequinox.scr.waitTimeOnBlock=60000 \
    -Dfelix.fileinstall.active.level=4 \
    -Djava.awt.headless=true \
    -cp $cp org.eclipse.equinox.launcher.Main $* \
    -console
```

- symlink jython.jar into /opt/openhab/lib/ (mkdir /opt/openhab/lib; cd /opt/openhab/lib; ln -s /opt/jython/jython.jar .)  or add jython.jar to classpath in start-script

### Running jython scripts from within the IDE
First install jython (in e.g. /opt/jython) as above, then:
- In the IDE "Run Configurations" create your own custom runtime environment by right-clicking "openHAB Runtime" and copying it into "openHAB Runtime Custom"
- In the "openHAB Runtime Custom", in the "Main" tab, add /opt/jython/jython.jar into the Boostrap entries.
- In the "Arguments" tab, add -Dpython.home=/opt/jython at the beginning of the "VM arguments"
- In the "Plug-ins" tab, verify that org.openhab.core.jsr223 has Auto-Start = true, and that "Start Level" is set larger than the default. So if the default Start Level is 4, you must set the Start Level of the jsr223 engine  to 5 to defer its start. **This step is crucially important to avoid null pointer exceptions and a broken system**.
- Now you can start the customized runtime by clicking on the down arrow next to the green start button, and selecting "openHAB Runtime Custom"

## Scripts
Each Script needs to be located in configurations/scripts with a correct script ending (".py", ".jy" for jython interpreter). Each Script can contain multiple rules.

### Rule-Class
Each rule is basically a class in the given scripting language. It needs to implement two functions:
* getEventTrigger
    * returns an array of EventTrigger
* execute
    * gets as first argument the reason why it was called

Supported triggers: 
* ChangedEventTrigger (for updates and changed states)
* CommandEventTrigger
* ShutdownTrigger
* StartUpTrigger
* TimerTrigger

```
class TestRule(Rule):
	def getEventTrigger(self):
		return [
			StartupTrigger(),
			ChangedEventTrigger("Heating_FF_Child", None, None),
			TimerTrigger("0/50 * * * * ?")
		] 
		
	def execute(self, event):
		oh.logDebug(self.getName(), "event received " + str(event))
		oh.logInfo(self.getName(), str(ItemRegistry.getItem("Heating_GF_Corridor")))
		action = oh.getActions() 
		oh.logInfo(self.getName(), "available actions: " + str(action.keySet()))
		ping = oh.getAction("Ping")
		oh.logInfo(self.getName(), "internet reachable: " + ("yes" if ping.checkVitality("google.com", 80, 100) else "no"))

def getRules():
    return RuleSet([
        TestRule()
    ])
```

#### Output
```
21:47:06.975 [DEBUG] [.openhab.model.jsr232.TestRule:60   ] - event received Event [triggerType=STARTUP, item=null, oldState=null, newState=null, command=null]
21:47:06.977 [INFO ] [.openhab.model.jsr232.TestRule:75   ] - Heating_GF_Corridor (Type=SwitchItem, State=OFF)
21:47:06.979 [INFO ] [.openhab.model.jsr232.TestRule:75   ] - available actions: [Exec, Transformation, Ping, HTTP, Audio]
21:47:07.003 [INFO ] [.openhab.model.jsr232.TestRule:75   ] - internet reachable: yes
```

## Interaction with Openhab
In general all interaction is done through the oh object. It has support for the following:

### Logging
* oh.logDebug(logger_name, format, arg0,....)
* oh.logInfo(logger_name, format, arg0,....)
* oh.logWarn(logger_name, format, arg0,....)
* oh.logError(logger_name, format, arg0,....) 

### BusEvent
* be.sendCommand([Item] item, [String] commandString)
* be.sendCommand([Item] item, [Numer] number)
* be.sendCommand([String] itemName, [String] commandString)
* be.sendCommand([Item] item, [Command] command)
* be.postUpdate([Item] item, [String] stateAsString)
* be.postUpdate([String] itemName, [String] stateAsString)
* be.postUpdate([String] itemName, [State] state)
* be.storeStates([Item[]] items)
* be.restoreStates([Map<Item, State>] statesMap)

### ItemRegistry
* ir.getItem(itemName) or ir.getItem(itemName)
* ir.getItemByPattern(String name)
* ir.getItems()
* ir.getItems(String pattern)
* ir.isValidItemName(String itemName)

### PersistenceExtensions
* pe.persist(Item item [, String serviceName]) or PersistenceExtensions.persist(Item item)
* pe.historicState(Item item, AbstractInstant timestamp [, String serviceName])
* pe.changedSince(Item item, AbstractInstant timestamp [, String serviceName])
* pe.updatedSince(Item item, AbstractInstant timestamp [, String serviceName])
* pe.maximumSince(Item item, AbstractInstant timestamp [, String serviceName])
* pe.minimumSince(Item item, AbstractInstant timestamp [, String serviceName])
* pe.averageSince(Item item, AbstractInstant timestamp [, String serviceName])
* pe.varianceSince(Item item, AbstractInstant timestamp [, String serviceName])
* pe.deviationSince(Item item, AbstractInstant timestamp [, String serviceName])
* pe.lastUpdate(Item item [, String serviceName])
* pe.deltaSince(Item item, AbstractInstant timestamp [, String serviceName])
* pe.evolutionRate(Item item, AbstractInstant timestamp [, String serviceName])
* pe.previousState(Item item)
* pe.previousState(Item item, boolean skipEqual)
* pe.previousState(Item item, boolean skipEqual [, String serviceName])

### Accessing actions
To access the old known methods (all actions) the openhab interface supports getActions() and getAction(name_of_action_provider).

```
oh.getActions()
ping = oh.getAction("Ping")
ping.checkVitality("google.com", 80, 100)
```

## Global variables/classes
##### OpenHab classes
* RuleSet
* Rule
* Command
* ChangedEventTrigger
* UpdatedEventTrigger
* CommandEventTrigger
* Event
* EventTrigger
* ShutdownTrigger
* StartupTrigger
* TimerTrigger
* TriggerType
* ItemRegistry (shortcut: "ir")
* BusEvent (shortcut: "be") 
* PersistenceExtensions (shortcut: "pe") 
* oh
* State
* HistoricItem

##### OpenHab Type-classes
* CallType
* DateTimeType
* DecimalType
* HSBType
* IncreaseDecreaseType
* OnOffType
* OpenClosedType
* PercentType
* PointType
* StopMoveType
* UpDownType
* StringType
* UndefType

##### Java classes
* DateTime
* StringUtils
* URLEncoder
* FileUtils
* FilenameUtils
* File 


# Examples

### Example in Jython
Open-Window on temperature (uses string casts, item info can also directly be obtained by the member method (intValue() etc.)) :
```
    import datetime

    class WintergartenRule(Rule):
        def __init__(self):
            item = ItemRegistry.getItem("Wintergarten_Automatik")
            if item.state == None or str(item.state) == "Uninitialized":
                oh.postUpdate("Wintergarten_Automatik", "ON")
                oh.postUpdate("Wintergarten_Automatik_from", "8")
                oh.postUpdate("Wintergarten_Automatik_to", "19")
                oh.postUpdate("Wintergarten_Automatik_openTemp", "23")
                oh.postUpdate("Wintergarten_Automatik_closeTemp", "20")

            self.logger = oh.getLogger("WintergartenRule")

        def getAutomatikInfo(self):
            return {
                "enabled": ItemRegistry.getItem("Wintergarten_Automatik").state == OnOffType.ON,
                "from": int(str(ItemRegistry.getItem("Wintergarten_Automatik_from").state)),
                "to": int(str(ItemRegistry.getItem("Wintergarten_Automatik_to").state)),
                "openTemp": float(str(ItemRegistry.getItem("Wintergarten_Automatik_openTemp").state)),
                "closeTemp": float(str(ItemRegistry.getItem("Wintergarten_Automatik_closeTemp").state))
            }

        def getEventTrigger(self):
            return [
                ChangedEventTrigger("Wintergarten_Temperatur")
            ]

        def execute(self, event):
            self.logger.info("event {}", event)
            if event.newState:
                self.logger.info("Wintergarten_Temperatur changed to: {}", event.newState)
				
                //PersistenceExtensions
                peExample = pe.historicState( ir.getItem("Wintergarten_Temperatur"), DateTime.now().minusDays(7))
                self.logger.info("Wintergarten_Temperatur last Week: {}", peExample)

                temp = float(str(event.newState))
                self.checkConditions(temp)

        def checkConditions(self, temp):
            config = self.getAutomatikInfo()

            t = datetime.datetime.now().time()

            if config["enabled"] and t < datetime.time(config["to"]) and t >= datetime.time(config["from"]):

                alleFenster = ItemRegistry.getItem("Wintergarten_Jalousie_Fenster_Alle")
                state = int(str(alleFenster.state))

                regen = str(ItemRegistry.getItem("Regensensor").state) == "1"
                print "Regen=",regen

                if not regen:
                    if temp >= config["openTemp"] and state == 100:
                        oh.sendCommand("Wintergarten_Jalousie_Fenster_Alle", "UP")
                    elif temp <= config["closeTemp"] and state <= 50:
                        oh.sendCommand("Wintergarten_Jalousie_Fenster_Alle", "DOWN")


    def getRules():
        return RuleSet([
            WintergartenRule()
        ])
```



### Example in JavaScript
```
'use strict';

//
//https://wiki.openjdk.java.net/display/Nashorn/Nashorn+extensions
// load Java Objects
//var anArrayList = new Java.type("java.util.ArrayList")
// or
//var ArrayList = Java.type("java.util.ArrayList");			

//load("foo.js"); 						// loads script from file "foo.js" from current directory
//load("http://www.example.com/t.js"); 	// loads script file from given URL

print("\n################# E X A M P L E S ##################\n");
oh.getLogger("E X A M P L E S");

var actionsTest = new Rule(){
    getEventTrigger: function(){
        oh.logDebug("getEventTrigger", "self:"+this);
        return [
            new TimerTrigger("0/15 * * * * ?")
        ];
    },
    execute: function(event){

        var action = oh.getActions();
        oh.logInfo("execute:"+__LINE__, "available actions: " + action.keySet());

        var action = oh.getActions();
        var actionArr = Java.from(action.keySet());
        oh.logInfo("execute:"+__LINE__, "available actions: " + action.keySet());

        for (var i=0; i<actionArr.length; i++) {
             oh.logInfo("execute:"+__LINE__, "Action: " +  oh.getAction(actionArr[i]));
        }

        var exec = oh.getAction("Exec");
        var tran = oh.getAction("Transformation");
        var mail = oh.getAction("Mail");
        var ping = oh.getAction("Ping");
        var transform = oh.getAction("Transformation").static.transform;
        var http = oh.getAction("HTTP");
        var audi = oh.getAction("Audio");
        var xmpp = oh.getAction("XMPP");

        oh.logInfo("execute:"+__LINE__, "Exec: " +  exec);
        oh.logInfo("execute:"+__LINE__, "Transformation: " + tran);
        oh.logInfo("execute:"+__LINE__, "Mail: " + mail);
        oh.logInfo("execute:"+__LINE__, "Ping: " + ping);
        oh.logInfo("execute:"+__LINE__, "HTTP: " + http);
        oh.logInfo("execute:"+__LINE__, "Audio: " + audi);
        oh.logInfo("execute:"+__LINE__, "XMPP: " + xmpp);

        oh.logInfo("TestRule", "internet reachable: " + (ping.static.checkVitality("google.com", 80, 100) ? "yes" : "no"));
        oh.logInfo("TestRule", "internet reachable: " + tran.static.transform("EXEC", "ls configurations/scripts",""));
        oh.logInfo("TestRule", "transform EXEC: " + tran.static.transform("EXEC", "ping 192.168.0.20",""));
        oh.logInfo("TestRule", "transform EXEC: " + transform("EXEC", "ping 192.168.0.20",""));
        oh.logInfo("TestRule", "transform EXEC: " + transform("EXEC", "ls configurations/scripts",""));

    }
};

var ohExample = new Rule(){
    getEventTrigger: function(){
        return [

			// E X A M P L E   T R I G G E R S
            //new ChangedEventTrigger("Heating_GF_Corridor", OnOffType.OFF, OnOffType.ON),
            //new ChangedEventTrigger("Heating_GF_Corridor"),
            //new CommandEventTrigger("Heating_GF_Corridor", OnOffType.ON),
            //new EventTrigger(),
            //new ShutdownTrigger(),
            //new StartupTrigger(),
            //new TimerTrigger("0 0/15 * * * ?")

            new TimerTrigger("0/15 * * * * ?")
        ];
    },
    execute: function(event){

		// E X A M P L E S   L O G G I N G
		print("\n################# E X A M P L E S   L O G G I N G ##################\n");
        oh.logDebug("execute::"+__LINE__,"event received "+event);
        oh.logInfo("execute:"+__LINE__, ir.getItem("Temperature_GF_Kitchen").toString());
        oh.logInfo("execute:"+__LINE__, ir.getItem("Heating_GF_Corridor").toString());
        oh.logInfo("execute::DateTime:"+__LINE__, DateTime.now().minusMinutes(30));
		
		// E X A M P L E S   I T E M S
        var Temperature_GF_Kitchen = ir.getItem('Temperature_GF_Kitchen');
		be.postUpdate("Temperature_GF_Kitchen", getRanTemp());
		be.postUpdate("Temperature_GF_Kitchen", getRanTemp());
		be.sendCommand("Temperature_GF_Kitchen", getRanTemp());
		oh.logInfo("execute:"+__LINE__, Temperature_GF_Kitchen.toString());
        oh.logInfo("execute::item:"+__LINE__, Temperature_GF_Kitchen);
        oh.logInfo("execute::PersistenceExtensions :"+__LINE__, "Temperature_GF_Kitchen in the past: "+pe.changedSince(Temperature_GF_Kitchen,DateTime.now().minusMinutes(10)));

		var Heating_GF_Corridor = ir.getItem('Heating_GF_Corridor');
        if(Heating_GF_Corridor.state.toString() == "Uninitialized"){
			oh.logInfo("execute:"+__LINE__, "Heating_GF_Corridor is 'Uninitialized' "+Heating_GF_Corridor.toString());
		}
		be.postUpdate("Heating_GF_Corridor", randomIntFromInterval(0,1) < 1 ? "OFF":"ON");
    }
};

// E X A M P L E   T O   G E T   A L L   I T E M S
var autoOffRule = new Rule() {
    getEventTrigger: function() {
        return [
            new TimerTrigger("0/15 * * * * ?")
        ];
    }, 
    execute: function(event) {
        oh.logDebug("execute"+__LINE__,"autoOffRule");
		var allItems = ir.getItems();
        for each(var nextItem in allItems) {
            if (nextItem.getState() == OnOffType.ON) {
                var dt = DateTime.now().minusMinutes(5);
                print(" #### 1. nextItem changedSince:" + pe.changedSince(nextItem, dt));
                if (!(pe.changedSince(nextItem, dt))) {
                    print(" #### 2. Do something with " + nextItem.getName());
                }
            }else{
                print(" #### 3. nextItem Name: " + nextItem.getName());
            }
        }
    }
};

// H E L P E R S 
var getRanTemp = function(){
	return randomIntFromInterval(-20,40);
}
function randomIntFromInterval(min,max){
    return Math.floor(Math.random()*(max-min+1)+min);
}

// E N A B L E   R U L E S 
function getRules(){return new RuleSet([ohExample, actionsTest]);}
```

###Example in Groovy
```
import org.openhab.core.jsr223.internal.shared.*
import org.openhab.core.items.Item
import org.openhab.core.items.ItemRegistry
import org.openhab.core.persistence.extensions.PersistenceExtensions
import org.joda.time.DateTime

Global.itemRegistry = this.ItemRegistry
Global.pe = this.pe

class Global {
	static ItemRegistry itemRegistry
	static Class pe
}

class GroovyTestRule implements Rule {
	def logger = Openhab.getLogger('TestRule')

	java.util.List<StartupTrigger> getEventTrigger() {
		logger.debug('foo')
		return [
			new StartupTrigger()
//			new ChangedEventTrigger("Foo", null, null)
		]
	}

	void execute(Event event) {
		logger.debug('Event received: ' + event.toString())
		logger.debug('Actions: ' + Openhab.getActions())

		Openhab.sendCommand("Bar", "5")

		if(event.getItem() != null) {
			logger.debug('Got Item: ' + event.getItem())
			logger.debug('Trying to retrieve history state...')			
			logger.debug(Globals.pe.averageSince(event.getItem(), DateTime.now().minusMinutes(30)))			
		}
		Item item = Global.itemRegistry.getItem('Bar')
		logger.debug("averageSince(): " + Global.pe.averageSince(item, DateTime.now().minusMinutes(30)))
		logger.debug("changedSince(): " + Global.pe.changedSince(item, DateTime.now().minusMinutes(2)))
		logger.debug("maximumSince(): " + (Global.pe.maximumSince(item, DateTime.now().minusMinutes(30))).state)
	}
}

RuleSet getRules() {
	return new RuleSet(new GroovyTestRule())
}
```
