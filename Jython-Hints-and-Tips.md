# Debugging

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