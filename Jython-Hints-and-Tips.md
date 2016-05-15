<a name="top"></a>
This page contains ideas for using [Jython](http://www.jython.org/) with the [[JSR223 Script Engine]] in Openhab.

* [Using traceback for better stack traces](#traceback)
* [Exploring the Jython API](#exploring)
* [Fixing missing getRules() error](#missing-getrules)
* [Easier logging](#easier-logging)

<a name="traceback"></a>
## Using _traceback_ for better stack traces

Without help, the JSR223 engine simply prints a Java stack trace, from which it is difficult to determine the error.  Using the Python traceback module gives useful error indications.

```python
import traceback

class TestRule(Rule):
    def getEventTrigger(self):
        return [
            TimerTrigger("0/50 * * * * ?")
        ]

    def execute(self, event):
        try:
            self.foo = unknown_bar  # error
        except:
            oh.logError("TestRule", traceback.format_exc())
```

[Top](#top)

<a name="exploring"></a>
## Exploring the openHAB Jython API

If you need to know the specific interface for a openHAB-specific type, you can create a simple Jython script to log some information. A Jython script isn't required to contain rules (although it *must* contain a `getRules` function even if it returns an empty `RuleSet`). Any top-level statements will be evaluated by Jython. Let's say you want to know what functions can be called on an openHAB Item.

### Example items/test.items

```
String TestString
```

### Example scripts/test.py

```python
item = ir.getItem("TestString")
oh.logInfo("TEST", str(type(item)))
oh.logInfo("TEST", str(dir(item)))

# no rules in this script
def getRules():
  return RuleSet([])
```

**Note:** The arguments to openHAB's log methods must be a string. The standard Python `dir` function returns a string *list* so `str` is used to format it as a string. The standard Python `type` function returns a Java type wrapper (in Jython) that also must be formatted as a string.

### Log file output

```
2016-05-15 16:46:08.120 [INFO ] [o.c.j.i.e.scriptmanager.Script] - Loading Script test.py
2016-05-15 16:46:08.126 [INFO ] [o.c.j.i.e.scriptmanager.Script] - EngineName: jython
2016-05-15 16:46:08.705 [INFO ] [org.openhab.model.jsr223.TEST ] - <type 'org.openhab.core.library.items.StringItem'>
2016-05-15 16:46:08.713 [INFO ] [org.openhab.model.jsr223.TEST ] - ['__class__', '__copy__', '__deepcopy__', '__delattr__', 
'__doc__', '__ensure_finalizer__', '__eq__', '__format__', '__getattribute__', '__hash__', '__init__', '__ne__', '__new__', 
'__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__str__', '__subclasshook__', '__unicode__', 
'acceptedCommandTypes', 'acceptedDataTypes', 'addStateChangeListener', 'class', 'dispose', 'equals', 
'eventPublisher', 'getAcceptedCommandTypes', 'getAcceptedDataTypes', 'getClass', 'getGroupNames', 
'getName', 'getState', 'getStateAs', 'groupNames', 'hashCode', 'initialize', 'name', 'notify', 
'notifyAll', 'removeStateChangeListener', 'setEventPublisher', 'setState', 'state', 'toString', 'wait']
```

**Note:** In Jython, any Java method that starts with "get" and has no arguments can be accessed like a Python property. So `state = item.getState()` becomes `state = item.state`. Similarly, any Java method that starts with "set" and has one argument can be accessed like Python property. So `item.setState(value)` becomes `item.state = value`.

[Top](#top)

<a name="missing-getrules"></a>
## Script file misses mandatory function: _getRules()_

This error appears to be caused by the Java file reader reading the file faster than the code is written by the editor.  I have removed its occurrence by updating _openhab.cfg_ to not read the script files as quickly

```
folder:items=10,items
folder:sitemaps=10,sitemap
folder:rules=10,rules
folder:scripts=50,script
folder:persistence=10,persist
```

[Top](#top)

<a name="easier-logging"></a>
## Easier Logging
Creating a super class that automatically creates a logger with using the rule class name.  Note that TestRule inherits from OpenhabRule, and calls its parents initializer.

```python
class OpenhabRule(Rule):
    def __init__(self):
        self.log = oh.getLogger(type(self).__name__)

class TestRule(OpenhabRule):
    def __init__(self):
        super(TestRule, self).__init__()

    def getEventTrigger(self):
        return [
            TimerTrigger("0/50 * * * * ?")
        ]


    def execute(self, event):
        self.log.debug("event received: {}", event)
```

[Top](#top)
