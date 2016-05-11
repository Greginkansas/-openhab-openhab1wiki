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