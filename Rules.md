How to work with automation rules

* [Introduction] (Rules#introduction)
* [Defining Rules] (Rules#defining-rules)
  * [File Location] (Rules#file-location)
  * [IDE Support] (Rules#ide-support)
  * [The Syntax] (Rules#the-syntax)
  * Triggers
    * [Rule Triggers] (Rules#rule-triggers)
    * [Item- / Event-based Triggers] (Rules#item---event-based-triggers)
    * [Time-based Triggers] (Rules#time-based-triggers)
    * [System-based Triggers] (Rules#system-based-triggers)
  * [Implicit Variables inside the Execution Block] (Rules#implicit-variables-inside-the-execution-block)
  * [Using the same Variable name across separate rule files] (Rules#using-the-same-variable-name-across-separate-rule-files)
  * [Concurrency Guard] (Rules#concurrency-guard)
  * [Logging] (Rules#logging)
  * [Example] (Rules#example)

## Introduction

"Rules" are used for automating processes: Each rule can be triggered, which invokes a script that performs any kinds of tasks, e.g. turn on lights by modifying your items, do mathematical calculations, start timers etcetera (for more details on _what_ a rule can do, see [scripts documentation](Scripts)).

openHAB has a highly integrated, lightweight but yet powerful rule engine included.
On this page you will learn how to leverage its functionality to do *real* home automation.

## Defining Rules

### File Location

Rules are placed in the folder `${openhab.home}/configurations/rules`. The runtime already comes with a demo file called `demo.rules`, which has a couple of examples, which can be a good starting point.

A rule file can contain multiple rules. All rules of a file share a common execution context, i.e. they can access and exchange variables with each other. It therefore makes sense to have different rule files for different use-cases or categories.

### IDE Support

The openHAB Designer offers full IDE support for rules, which includes syntax checks and coloring, validation with error markers, content assist (Ctrl+Space) incl. templates etc. This makes the creation of rules very easy!

![](images/screenshots/designer-rules.png)

### The Syntax

Note: The rule syntax is based on [Xbase](http://www.eclipse.org/Xtext/#xbase) and as a result it is sharing many details with [Xtend](http://www.eclipse.org/xtend/), which is built on top of Xbase as well. As a result, we will often point to the Xtend documentation for details.

A rule file is a text file with the following structure:

    [Imports]
    
    [Variable Declarations]
    
    [Rules]


The ***Imports*** section contains import statement just like in Java. As in Java, they make the imported types available without having to use the fully qualified name for them. For further details, please see the [Xtend documentation for imports](http://www.eclipse.org/xtend/documentation/202_xtend_classes_members.html#imports).

Example:
```Xtend
    import org.openhab.core.library.types.*
```
The ***Variable Declarations*** section can be used to declare variables that should be accessible to all rules in this file. You can declare variables with or without initial values and modifiable or read-only. For further details, please see the [Xtend documentation for variable declarations](http://www.eclipse.org/xtend/documentation/203_xtend_expressions.html#variable-declaration).

Example: 
```Xtend
    // a variable with an initial value. Note that the variable type is automatically inferred
    var counter = 0 
    
    // a read-only value, again the type is automatically inferred
    val msg = "This is a message" 
    
    // an uninitialized variable where we have to provide the type (as it cannot be inferred from an initial value)
    var Number x 
```
The ***Rules*** section contains a list of rules. Each rule has the following syntax:
```Xtend
    rule "rule name"
    when
        <TRIGGER CONDITION1> or 
        <TRIGGER_CONDITION2> or 
        <TRIGGER_CONDITION3>
        ...
    then
        <EXECUTION_BLOCK>
    end
```
A rule can have any number of trigger conditions, but must at least have one.
The _EXECUTION_BLOCK_ contains the code that should be executed, when a trigger condition is met. The content of the _EXECUTION_BLOCK_ is in fact a script, so please refer to the [scripts documentation](Scripts) for details.

### Rule Triggers

Before a rule starts working, it has to be triggered.

There are different categories of rule triggers:
- **Item**(-Event)-based triggers: They react on events on the openHAB event bus, i.e. commands and status updates for items
- **Time**-based triggers: They react at special times, e.g. at midnight, every hour, etc.
- **System**-based triggers: They react on certain system statuses.

Here are the details for each category:

### Item- / Event-based Triggers

You can listen to commands for a specific item, on status updates or on status changes (an update might leave the status unchanged). You can decide whether you want to catch only a specific command/status or any. Here is the syntax for all these cases (parts in square brackets are optional):
```Xtend
    Item <item> received command [<command>]
    Item <item> received update [<state>]
    Item <item> changed [from <state>] [to <state>]
```

> A simplistic explanation of the differences between `command` and `update` (useful if you are new to openHAB) can be found on the [Actions page](https://github.com/openhab/openhab/wiki/Actions#sendcommand-vs-postupdate) 

### Time-based Triggers

You can either use some pre-defined expressions for timers or use a [cron expression](http://www.quartz-scheduler.org/documentation/quartz-2.1.x/tutorials/tutorial-lesson-06) instead:
```Xtend
    Time is midnight
    Time is noon
    Time cron "<cron expression>"
```
A cron expression takes the form of six or optionally seven fields:

1. Seconds
1. Minutes
1. Hours
1. Day-of-Month
1. Month
1. Day-of-Week
1. Year (optional field)

for more information see: http://www.quartz-scheduler.org/documentation/quartz-2.1.x/tutorials/tutorial-lesson-06

You may also use [CronMaker](http://www.cronmaker.com/) to generate cron expressions. 

### System-based Triggers

Currently, you schedule rules to be executed either at system startup or shutdown. Note that newly added or modified startup rules are executed once, even if openHAB is already up and running. They are simply executed once as soon as the system is aware of them. Here's the syntax for system triggers:
```Xtend
    System started
    System shuts down
```
## Implicit Variables inside the Execution Block

Besides the implicitly available variables for items and commands/states (see the [script documentation](Scripts)), rules can have additional pre-defined variables, depending on their triggers:

- Every rule that has at least one command event trigger, will have the variable `receivedCommand` available, which can be used inside the execution block.
- Every rule that has at least one status change event trigger, will have the variable `previousState` available, which can be used inside the execution block.
- The variable `now`, which is a DateTime object that represents the current time

## Using the same Variable name across separate rule files

It is worth being aware that there "could" be issues (despite what the documentation says) when using the same variable name in separate rule files. 

Taken from [1121](https://github.com/openhab/openhab/issues/1121) Developer Kai says:

> The theory is: Variables are global to all rules within one rule file and they are NOT visible to rules in other files. So it should be fine to use the same name for variables in different files.
> In practice, openHAB has afaik a bug regarding this and there actually ARE side effects if you use the same variable name across different files.
> Note that this has been fixed in ESH (https://bugs.eclipse.org/bugs/show_bug.cgi?id=423532), so in openHAB 2.0, this problem should not exist anymore.

## Concurrency Guard
If a rule triggers on UI events it may be necessary to guard against concurrency.
```Xtend
	import java.util.concurrent.locks.ReentrantLock

	var java.util.concurrent.locks.ReentrantLock lock  = new java.util.concurrent.locks.ReentrantLock()

	rule ConcurrentCode
	when
		Item Dummy received update
	then
		lock.lock()
		try {
			// do stuff (e.g. create and start a timer ...)
		} finally{
		   lock.unlock()
		}
	end
```
## Logging

You can emit log messages from your rules to aid debugging. There are a number of logging methods available from your rules, the java signatures are:
```java
    logDebug(String loggerName, String format, Object... args)
    logInfo(String loggerName, String format, Object... args)
    logWarn(String loggerName, String format, Object... args)
    logError(String loggerName, String format, Object... args)
```

In each case, the `loggerName` parameter is combined with the string `org.openhab.model.script.` to create the log4j logger name. For example, if your rules file contained the following log message:

```java
    logDebug("kitchen", "Kitchen light turned on")
```

then the logger you would have to enable in your logbook.xml file to allow that message to appear in your logs would be:

```xml
    <logger name="org.openhab.model.script.kitchen" level="DEBUG"/>
```

## Example

Taking all the information together, an example rule file could look like this:
```Xtend
    // import the decimal type as we refer to it in a rule
    import org.openhab.core.library.types.DecimalType
    
    var Number counter
    
    // setting the counter to some initial value
    // we could have done this in the variable declaration already
    rule Startup
    when 
    	System started
    then
    	counter = 0
    end
    
    // increase the counter at midnight
    rule "Increase counter"
    when
    	Time cron "0 0 0 * * ?"
    then
    	counter = counter + 1
    end
    
    // tell the number of days either at noon or if a button is pressed
    rule "Announce number of days up"
    when
    	Time is noon or 
    	Item AnnounceButton received command ON
    then
    	say("The system is up since " + counter + " days")
    end
    
    // sets the counter to the value of a received command
    rule "Set the counter"
    when
    	Item SetCounterItem received command
    then
    	counter = receivedCommand as DecimalType
    end
```
You can find further examples in the [openHAB-samples](https://github.com/openhab/openhab/wiki/Samples-Rules) section.