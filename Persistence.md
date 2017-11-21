## Introduction

Persistence support stores item states over time (a time series). openHAB is not restricted to a single data store. Different stores can co-exist and be configured independently.

When persisting states, there are many different possibilities one might think of: relational databases, NoSQL databases, round-robin databases, Internet-of-Things (IoT) cloud services, simple log files etc.
openHAB tries to make all of these options possible and configurable in the same way. Note that some options might only be good for exporting data (e.g. IoT services or log files), while others can be easily queried as well, and hence, may be used for providing historical data for openHAB functionality.

## Persistence Services

In openHAB, we call these different options the available "persistence services."  The table below lists those that are currently available.

Name | Queryable | Link | Notes
-----|:---------:|------|------
[[Cosm\|Cosm-Persistence]] | | [cosm.com](http://cosm.com/) | write item states to Cosm/Xively site
[[db4o\|db4o-Persistence]] | X | [db4o.com](http://www.db4o.com/) | a lightweight, 100% pure Java object database
[[Amazon DynamoDB\|Amazon-DynamoDB-Persistence]] | X | [amazon.com](https://aws.amazon.com/documentation/dynamodb/) | fully managed NoSQL database service from Amazon
[[Exec\|Exec-Persistence]] | | | persist by executing commands in the underlying OS
IFTTT |  | [IFTTT](https://ifttt.com/recipes) | if-this-then-that cloud service. See [[my.openHAB|my.openHAB-Persistence]]
[[InfluxDB\|InfluxDB-Persistence]] | X | [influxdata.com](https://influxdata.com/) | open-source distributed time series database with no external dependencies
[[JDBC\|JDBC-Persistence]] | X | [wikipedia](https://en.wikipedia.org/wiki/Java_Database_Connectivity) | Java Database Connectivity support for MySQL and several other JDBC-enabled databases (one table per item)
[[JPA\|JPA-Persistence]] | X | [wikipedia](http://en.wikipedia.org/wiki/Java_Persistence_API) | Java Persistence API support for Derby and many other JDBC-enabled databases (one table "historic_item" where all item states are stored as strings)
[[Logging\|Logging-Persistence]] | | [logback.qos.ch](http://logback.qos.ch/) | write item states to log files with a flexible format
[[MapDB\|mapdb-Persistence]] | X | [mapdb.org](http://www.mapdb.org/) | only saves the last item state; useful for restoreOnStartup strategy 
[[MongoDB\|MongoDB-Persistence]] | | [mongodb.com](https://www.mongodb.com/) | NoSQL document-oriented database
[[MQTT\|MQTT-Persistence]] | | [wikipedia](https://en.wikipedia.org/wiki/MQTT) | stores item states to an MQTT broker
[[my.openHAB\|my.openHAB-Persistence]] | | [my.openHAB.org](https://my.openhab.org/) | send item states to openHAB's cloud service, and use openHAB items in [IFTTT recipes](https://ifttt.com/recipes) -> outdated as of 31st Jan 2017
[[openHAB Cloud Connector\|openHAB-Cloud-Connector]] | | [myopenHAB.org](https://myopenhab.org/) | send item states to openHAB's cloud service, and use openHAB items in [IFTTT recipes](https://ifttt.com/recipes)
[[MySQL\|MySQL-Persistence]] | X | [mysql.com](https://www.mysql.com/) | one SQL table per item
[[RRD4J\|rrd4j-Persistence]] | X | [RRD4J](https://github.com/rrd4j/rrd4j) | Java version of the powerful round-robin database [RRDtool](http://oss.oetiker.ch/rrdtool/).  Numeric states only.
[[Sen.se\|Sense-Persistence]] | | [open.sen.se](http://open.sen.se/) | send item states to the Sen.Se web site
[[SiteWhere\|SiteWhere-Persistence]] | | [SiteWhere](http://www.sitewhere.org/) | send item states to a SiteWhere server instance running locally or in the cloud
Xively | | [cosm.com](http://cosm.com/) | See [[Cosm\|Cosm-Persistence]]

## Configuration

For every persistence service that you have installed and want to use, you have to put a configuration file named `<persistenceservice>.persist` (e.g. `db4o.persist`) in the folder `${openhab.home}/configurations/persistence`. These files should be edited with the openHAB Designer, which provides syntax checks, auto-completion and more.

Before going into the details of the syntax of these files, let us discuss the concept behind it first:
The basic idea is to provide a simple way to tell openHAB, which items should be persisted and when. The persistence configuration defines so called "strategies" for this. These are very similar to the triggers of [[Rules|rules]] as you will most likely also either persist a value when some bus event occurred (i.e. an item state has been updated or changed) or with a fixed schedule as the [cron expressions](http://www.quartz-scheduler.org/documentation/quartz-2.1.x/tutorials/crontrigger) allow to define. 

Persistence configuration files consist of several sections:

**Strategies section**: This section allows you to define strategies and to declare a set of default strategies to use (for this persistence service). The syntax is as follows:
```
Strategies {
    <strategyName1> : "<cronexpression1>"
    <strategyName2> : "<cronexpression2>"
    ...
    
    default = <strategyNameX>, <strategyNameY>
}
```

Note that a strategies section must be included (with a default defined), or the persistence services will not work.

The following strategies are already statically defined (and thus do not need to be listed here, but can be declared as a default):
* everyChange: persist the state whenever its state has changed
* everyUpdate: persist the state whenever its state has been updated, even if it did not change
* restoreOnStartup:If the state is undefined at startup, the last persisted value is loaded and the item is initialized with this state. This is very handy for "virtual" items that do not have any binding to real hardware, like "Presence" or similar.

**Items section**: This defines, which items should be persisted with which strategy. The syntax is:
```
Items {
    <itemlist1> [-> "<alias1>"] : [strategy = <strategy1>, <strategy2>, ...]
    <itemlist2> [-> "<alias2>"] : [strategy = <strategyX>, <strategyY>, ...]
    ...
}
```
where `<itemlist>` is a comma-separated list of the following options:
- `*` - this line should apply to all items in the system.
- `<itemName>` - a single item identified by its name. This item can be a group item, but note that only its own (group) value will be persisted, not the states of its members.
- `<groupName>*` - all members of this group will be persisted, but not the group itself.
  If no strategies are provided, the default strategies that are declared in the first section are used.
  Optionally, an alias can be provided, if the persistence service requires special names (e.g. a table to use in a database, a feed id for an IoT-service etc.) 

A valid persistence configuration file might look like this:

    // persistence strategies have a name and a definition and are referred to in the "Items" section
    Strategies {
    	everyHour : "0 0 * * * ?"
    	everyDay  : "0 0 0 * * ?"
    
    	// if no strategy is specified for an item entry below, the default list will be used
    	default = everyChange
    }
    
    /* 
     * Each line in this section defines for which item(s) which strategy(ies) should be applied.
     * You can list single items, use "*" for all items or "groupitem*" for all members of a group
     * item (excl. the group item itself).
     */
    Items {
    	// persist all items once a day and on every change and restore them from the db at startup
    	* : strategy = everyChange, everyDay, restoreOnStartup
    	
    	// additionally, persist all temperature and weather values every hour
    	Temperature*, Weather* : strategy = everyHour
    }

## Persistence Extensions in Scripts and Rules

To make use of persisted states inside scripts and rules, a couple of useful extensions have been defined on items. In contrast to an action (which is a function that can be called anywhere in a script or rule), an extension is a function that is only available like a method on a certain type. This means that the persistence extensions are available like methods on all items. An example will make this clearer:
The statement

    Temperature.historicState(now.minusDays(1))
will return the state of the item "Temperature" from 24 hours ago. You can easily imagine that you can implement very powerful rules with this kind of feature.

Here is the full list of available persistence extensions:

    <item>.persist - Persists the current state
    <item>.lastUpdate - Query for the last update timestamp of a given item.
    <item>.historicState(AbstractInstant) - Retrieves the historic item at a certain point in time
    <item>.changedSince(AbstractInstant) - Checks if the state of the item has (ever) changed since a certain point in time
    <item>.updatedSince(AbstractInstant) - Checks if the state of the item has been updated since a certain point in time
    <item>.maximumSince(AbstractInstant) - Gets the Item with the maximum value (state) since a certain point in time
    <item>.minimumSince(AbstractInstant) - Gets the Item with the minimum value (state) since a certain point in time
    <item>.averageSince(AbstractInstant) - Gets the average value of the state of a given item since a certain point in time.
    <item>.deltaSince(AbstractInstant) - Gets the difference value of the state of a given item since a certain point in time.
    <item>.previousState() - Retrieves the previous item (returns HistoricItem).
    <item>.previousState(true) - Retrieves the previous item, skips items with equal state values and searches the first item with state not equal the current state (returns HistoricItem).
    <item>.sumSince(AbstractInstant) - Retrieves the sum of the previous states since a certain point in time. (OpenHab 1.8)

These extensions use the default persistence service that is configured in openhab.cfg. If the default service should not be used, all extensions accept a String as a further parameter for the persistence service to use (e.g. "rrd4j" or "sense").

For all kinds of time and date calculations, [Jodatime](http://joda-time.sourceforge.net/) has been made available in the scripts and rules. This makes it very easy to navigate through time, here are some examples of valid expressions:

    Lights.changedSince(now.minusMinutes(2).minusSeconds(30))
    Temperature.maximumSince(now.toDateMidnight)
    Temperature.minimumSince(parse("2012-01-01"))
    PowerMeter.historicState(now.toDateMidnight.withDayOfMonth(1))
The "now" variable can be used for relative time expressions, while "parse()" can define absolute dates and times. See the [Jodatime documentation](http://joda-time.sourceforge.net/api-release/org/joda/time/format/ISODateTimeFormat.html#dateTimeParser()) on what string formats are supported for parsing.

## Startup Behavior
Persistence services and the rule engine are started in parallel. It is possible that rules may execute using items that have not had their persisted state restored yet (In this case, these items have an "undefined" state at the time the rule is executed). Therefore, rules that rely on persisted information may break during startup.

#### Workaround 1

A workaround which helps in some cases is to introduce an item e.g. "delayed_start" that is set to "OFF" at startup and to "ON" some time later (when it can be assumed that persistence has restored all items. The time needs to be determined empirically. It is influenced by the size of your home automation project and the performance of your platform).
The affected rules then have to be masked by using "delayed_start".

#### Workaround 2
Create `configurations/rules/refresh.rules` with following content. It runs a refresh script when openHAB is started.

    var boolean reloadOnce = true
    rule "Refresh rules after persistance service started"
      when System started
    then
      if(reloadOnce)
        executeCommandLine("./configurations/rules_refresh.sh")
      else
        println("reloadOnce is false")
      reloadOnce = false
    end

Create a refresh script `configurations/rules_refresh.sh` and make it executable (`chmod +x rules_refresh.sh`):

    # This script is called by openHAB after the persistence service has started
    sleep 5
    cd [full_path_to_openhab]/configurations/rules/
    RULES=`find *.rules | grep -v refresh.rules`
    for f in $RULES
    do
      touch $f
    done 

The script waits for 5 seconds, then touches all `*.rules` files (except `refresh.rules`) which causes openHAB to reload these files. Other rules-files may be added on new lines.  (As noted above, you will have to experiment to find the appropriate sleep value for your specific system.)