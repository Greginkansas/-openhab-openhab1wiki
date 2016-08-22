**Note**: Currently this scriptengine cannot be used in OpenHAB2. A new smarthome compatible functionality is in development. Scripts from this engine can be used in the new addon without significant changes. A compatibility layer will be provided.


## Installation

For Jython scripting, use a 1.8.x build.

### Install Jython

Download Jython 2.7.0 from http://www.jython.org/downloads.html as an installer package.

Install Jython to `/opt/jython`. The package can be installed to other directories (on Windows, for example), but the following instructions will need to use that path for the `python.home` configuration in the startup script.

Customize your startup script (start.sh, for example) to reference to the Jython installation directory. This is necessary so Jython can access its library files:

The startup script details will vary depending on the way openHAB was installed (apt-get, manual, etc.). However, the important changes are to add `-Dpython.home=/opt/jython` to the Java command line and to make sure the jython.jar file is in the Java classpath.

Create a symbolic link to the jython.jar file in /opt/openhab/lib/ or directly add the full path to the jython.jar file to the classpath in the startup script.

#### Linking jython.jar

```
# paths may differ depending on Jython and openHAB installation locations
mkdir /opt/openhab/lib
cd /opt/openhab/lib
ln -s /opt/jython/jython.jar .
```

#### Configuring the Python module import path

It is **recommended** that you add `-Dpython.path="configurations/scripts/lib"` to your startup arguments. You can than create library scripts in scripts/lib folder and then easily import them from you normal scripts. This makes creating library functions very easy! This suggested path assumes that the openHAB server directory is the configurations parent directory. If you have issues, try using an absolute path for `python.path`.

**Note:** There is nothing special about the "configurations/scripts/lib" directory. You can use any directory you'd like and even specify multiple directories separated by the path character on the host operating system (":" for Linux, ";" for Windows).

#### Example startup script

```
#!/bin/sh

cd `dirname $0`

# set path to eclipse folder. If local folder, use '.'; otherwise, use /path/to/eclipse/
eclipsehome="server";

# set ports for HTTP(S) server
HTTP_PORT=8080
HTTPS_PORT=8443

# Set this to the directory where Jython was installed
JYTHON_HOME="/opt/jython";

# get path to equinox jar inside $eclipsehome folder
classpath=$(echo lib/*.jar | tr ' ' ':'):$(find $eclipsehome -name "org.eclipse.equinox.launcher_*.jar" | sort | tail -1);
echo $classpath

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
    -cp $classpath org.eclipse.equinox.launcher.Main $* \
    -console
```

#### For apt-get installations on Linux
The apt-get packages run openhab as an executable jar file, so modifying classpath has no effect.  

To use Jython, update JAVA_ARGS in  /etc/default/openhab to this

```
JAVA_ARGS="-Xbootclasspath/a:/opt/jython/jython.jar -Dpython.home=/opt/jython -Dpython.path=configurations/scripts/lib"
```
## Scripts
Each Script needs to be located in configurations/scripts with a correct script ending (".py", ".jy" for jython interpreter). Each Script can contain multiple rules.

### Rules
Rules are implemented using Jython subclasses of the `Rule` interface. The class must implement two required functions:

* getEventTrigger(self)
    * returns an list of EventTrigger implementations
* execute(self, event)
    * the `event` argument has details about what triggered the execution

#### Example

```
class ExampleRule(Rule):
	def getEventTrigger(self):
		return [ StartupTrigger() ]
		
	def execute(self, event):
		oh.logInfo("ExampleRule", str(event))
```

## Interaction with openHAB

To faciliate Jython scripting, several openHAB classes are predefined (no Jython import is needed to use these classes). These classes include:

|Python Type|Notes|
|---------|-----|
|RuleSet| |
|Rule| Interface for Jython rules. |
|BusEvent| Aliased to 'be'. Superclass of `Openhab` |
|PersistenceExtensions| Aliased to 'pe' |
|HistoricItem| Used with PersistenceExtensions |
|Openhab| Aliased to 'oh'. Also has all the BusEvent functionality. |
|State| |
|Command| |
|ItemRegistry| Aliased to 'ir'. |
|DateTime| Joda date/time class. |
|StringUtils| org.apache.commons.lang.StringUtils |
|URLEncoder| java.net.URLEncoder |
|FileUtils| org.apache.commons.io.FileUtils |
|FilenameUtils| org.apache.commons.io.FilenameUtils |
|File| java.io.File |

**Note:** Several of these classes overlap with standard Python library functionality (file handling, string manipulation).

### Triggers

|Python Type|Notes|
|---------|-----|
|ChangedEventTrigger| Triggers on Item state changes |
|UpdatedEventTrigger| Triggers on Item state updates |
|CommandEventTrigger| Triggers on Item commands |
|ShutdownTrigger| Triggers when binding is shutdown |
|StartupTrigger| Triggers when rule is defined (or redefined) |
|TimerTrigger| Triggers based on a cron specification. |
|TriggerType| UPDATE, CHANGE, COMMAND, STARTUP, SHUTDOWN, TIMER |
|Event| Type passed to `execute(self, event)` callback. The Event class has the following attributes: triggerType, item, oldState, newState, command|

### State Types

- CallType
- DateTimeType
- DecimalType
- HSBType
- IncreaseDecreaseType
- OnOffType
- OpenClosedType
- PercentType
- PointType
- StopMoveType
- UpDownType
- StringType
- UnDefType


### Openhab (alias: oh)
- Logging
  - oh.logDebug(logger_name, format, arg0,....)
  - oh.logInfo(logger_name, format, arg0,....)
  - oh.logWarn(logger_name, format, arg0,....)
  - oh.logError(logger_name, format, arg0,....) 
- Events
  - oh.sendCommand([Item] item, [String] commandString)
  - oh.sendCommand([Item] item, [Numer] number)
  - oh.sendCommand([String] itemName, [String] commandString)
  - oh.sendCommand([Item] item, [Command] command)
  - oh.postUpdate([Item] item, [String] stateAsString)
  - oh.postUpdate([String] itemName, [String] stateAsString)
  - oh.postUpdate([String] itemName, [State] state)
  - oh.storeStates([Item[]] items)
  - oh.restoreStates([Map<Item, State>] statesMap)

### ItemRegistry (alias: ir)
* ir.getItem(String itemName)
* ir.getItemByPattern(String name)
* ir.getItems()
* ir.getItems(String pattern)
* ir.isValidItemName(String itemName)

### PersistenceExtensions (alias: pe)
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

### Accessing openHAB actions
To access openHAB actions, the interface supports getActions() and getAction(name_of_action_provider).

```
actions = oh.getActions()
oh.logInfo("MyRule", str(actions))
```

```
ping = oh.getAction("Ping")
ping.checkVitality("google.com", 80, 100)
```

<a name="jython-examples"></a>
## Examples

See [[Jython Hints and Tips]] page for more examples and suggestions.

### Event Triggers - Details and Example

```python
ChangedEventTrigger(itemName, fromState=None, toState=None)
```

Triggers when an item's state has changed to a different value. The fromState and toState arguments are optional and must be `State` objects. For example, you must specify OnOffType.ON and not "ON" for a state.
   
```
UpdatedEventTrigger(itemName)
```

Triggers when an item's state is set (could be same or different values than previous state).

```
CommandEventTrigger(itemName, command)
```

Triggers when the specified command is sent to an item. The command must be a `Command` object.

```
StartupTrigger()
```

Triggers when the rule is created (or redefined by a script file reload).

```
ShutdownTrigger()
```

Triggers when the binding is shutdown.

```
TimerTrigger(cronspec)
```

Triggers at times specified by the `cronspec`. For more information about cron expressions see the [Quartz documentation](http://www.quartz-scheduler.org/documentation/quartz-2.x/tutorials/tutorial-lesson-06.html).

#### Example Trigger Usage

```python
class TestRule(Rule):
	def getEventTrigger(self):
		return [
			StartupTrigger(),
			ChangedEventTrigger("Heating_FF_Child", None, None),
			TimerTrigger("0/50 * * * * ?")
		] 

	def getName(self):
		return type(self).__name__

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

### Shutter Control Example

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

## Other Resources

#### Tutorial

There is a Jython-related tutorial on the [community forum](https://community.openhab.org/t/tutorial-beyond-xtext-supercharged-rules-using-jython/3417).
 
#### Tips

See [[Jython hints and tips|Jython-Hints-and-Tips]] page for information about improved error logging, reuseable classes and so on.

### Jython Script Collection
On the [community forum](https://community.openhab.org/t/jsr223-script-collection/5232) some useful commonly usable Jython scripts can be found. The scripts themself can be found at the [Github Repository](https://github.com/smerschjohann/openhab-jsr223-scripts).

### Jython Utilities

Jython utilities and experimental code can be found [here](https://github.com/steve-bate/openhab-jython).


## Configuring the Eclipse IDE to run Jython scripts

First install jython (in e.g. /opt/jython) as above, then:
- In the IDE "Run Configurations" create your own custom runtime environment by right-clicking "openHAB Runtime" and copying it into "openHAB Runtime Custom"
- In the "openHAB Runtime Custom", in the "Main" tab, add /opt/jython/jython.jar into the Boostrap entries.
- In the "Arguments" tab, add -Dpython.home=/opt/jython at the beginning of the "VM arguments"
- In the "Plug-ins" tab, verify that org.openhab.core.jsr223 has Auto-Start = true, and that "Start Level" is set larger than the default. So if the default Start Level is 4, you must set the Start Level of the jsr223 engine  to 5 to defer its start. **This step is crucially important to avoid null pointer exceptions and a broken system**.
- Now you can start the customized runtime by clicking on the down arrow next to the green start button, and selecting "openHAB Runtime Custom"
