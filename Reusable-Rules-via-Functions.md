Quite often, there are rules with all the same logic, but operating on different items and with different parameters. Using [lambda expressions](https://eclipse.org/xtend/documentation/203_xtend_expressions.html#lambdas), it is in fact possible to specify reusable and parameterizable functions. The example below can be used to control rollershutters:
* **S_[name]_up** is a switch (input) item to move it up
* **S_[name]_down** is a switch (input) item to move it down
* **R_[name]_up** is the switch item that actually moves it up
* **R_[name]_down** is the switch item that actually moves it down

Some constants and functions are defined at the top:
```xtend
// total runtime of the rollershutters
val int ROLLO_TIMEOUT = 20

// timers to switch it off after the timeout
val java.util.Map<String, org.openhab.model.script.actions.Timer> rolloTimers = newHashMap

// lambda expression that can be used as a function (here: with 5 parameters)
val org.eclipse.xtext.xbase.lib.Functions$Function5 rolloLogic = [
	org.openhab.core.library.items.SwitchItem relayItem,
	org.openhab.core.library.items.SwitchItem relayItemOpposite,
	java.util.Map<String, org.openhab.model.script.actions.Timer> timers,
	String timerKey, int timeout |
		// if current state is off
		if (relayItem.state == OFF) {
			// if opposite state is also off
			if (relayItemOpposite.state == OFF) {
				// switch it on!
				relayItem.sendCommand(ON)
				// if there is already a timer, cancel it
				timers.get(timerKey)?.cancel
				// now create a new timer for switching off
				timers.put(timerKey, createTimer(now.plusSeconds(timeout)) [|
					// switch it off afterwards, if it is not already off
					if (relayItem.state == ON)
						relayItem.sendCommand(OFF)
					// remove timer from map because nothing is ON anymore
					timers.remove(timerKey)
				])
			} else { // opposite is on
				// switch opposite off!
				relayItemOpposite.sendCommand(OFF)
			}
		} else { // it is already on
			// switch it off
			relayItem.sendCommand(OFF)
		}
]
```
And the actual rules simply call the function `rolloLogic` with their specific parameters:
```xtend
rule "rollo kitchen up"
when Item S_Kitchen_up changed to ON then
	rolloLogic.apply(R_Kitchen_up, R_Kitchen_down, rolloTimers, "Kitchen", ROLLO_TIMEOUT)
end
rule "rollo kitchen down"
when Item S_Kitchen_down changed to ON then
	rolloLogic.apply(R_Kitchen_down, R_Kitchen_up, rolloTimers, "Kitchen", ROLLO_TIMEOUT)
end
rule "rollo bathroom up"
when Item S_Bathroom_up changed to ON then
	rolloLogic.apply(R_Bathroom_up, R_Bathroom_down, rolloTimers, "Bathroom", ROLLO_TIMEOUT)
end
rule "rollo bathroom down"
when Item S_Bathroom_down changed to ON then
	rolloLogic.apply(R_Bathroom_down, R_Bathroom_up, rolloTimers, "Bathroom", ROLLO_TIMEOUT)
end
```
This should simplify many rule definitions :-)