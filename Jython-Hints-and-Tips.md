This page contains ideas for using [Jython](http://www.jython.org/) with the [[JSR223 Script Engine]] in Openhab.

# Debugging

## Using _traceback_

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
## Script file misses mandotary function: _getRules()_
This error appears to be caused by the Java file reader reading the file faster than the code is written by the editor.  I have removed its occurrence by updating _openhab.cfg_ to not read the script files as quickly

```
folder:items=10,items
folder:sitemaps=10,sitemap
folder:rules=10,rules
folder:scripts=50,script
folder:persistence=10,persist
```
# Logging
Creating a super class that automatically creates a logger with the class name.  Note that TestRule inherits from OpenhabRule, and calls its parents initializer.

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