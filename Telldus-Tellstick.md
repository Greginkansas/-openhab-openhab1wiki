### Telldus Tellstick

Below is an example of how to capture events from Telldus Tellstick and forward them to the openhab bus as events through REST. The example works for on/off switches and wireless sensors but can easily be extended to dimmers, etc. Events are sent to openhab regardless if a telldus state change originates from a wireless remote or from other software (e. g. Switchking or Telldus Center). In other word, the script keeps openhab in sync with telldus states.

You need:
- Tellstick Duo (RF Transmitter/Receiver)
- Telldus Core software from here: http://developer.telldus.se/
- Tellcore Python lib: https://pypi.python.org/pypi/tellcore-py

See wiki section [Binding configurations](https://code.google.com/p/openhab-samples/wiki/BindingConfig?ts=1378067942&updated=BindingConfig#How_to_send_commands_to_Telldus_Tellstick) for the outbound example (openhab to tellstick).

Tested on Ubuntu Server 13.04.
```python
    #!/usr/bin/env python
    
    import sys
    import time
    
    import tellcore.telldus as td
    from tellcore.constants import *
    
    import httplib
    
    openhab = "localhost:8080"
    headers = {"Content-type": "text/plain"}
    connErr = "No connection to openhab on http://" + openhab
    
    METHODS = {TELLSTICK_TURNON: 'ON',
               TELLSTICK_TURNOFF: 'OFF',
               TELLSTICK_BELL: 'BELL',
               TELLSTICK_TOGGLE: 'toggle',
               TELLSTICK_DIM: 'dim',
               TELLSTICK_LEARN: 'learn',
               TELLSTICK_EXECUTE: 'execute',
               TELLSTICK_UP: 'up',
               TELLSTICK_DOWN: 'down',
               TELLSTICK_STOP: 'stop'}
    
    def raw_event(data, controller_id, cid):
        string = "[RAW] {0} <- {1}".format(controller_id, data)
        print(string)
    
    def device_event(id_, method, data, cid):
        method_string = METHODS.get(method, "UNKNOWN STATE {0}".format(method))
        string = "[DEVICE] {0} -> {1}".format(id_, method_string)
        if method == TELLSTICK_DIM:
            string += " [{0}]".format(data)
        print(string)
        url = "/rest/items/td_device_{0}/state".format(id_)
        try:
            conn = httplib.HTTPConnection(openhab)
            conn.request('PUT', url, method_string, headers)
        except:
            print(connErr)
    
    def sensor_event(protocol, model, id_, dataType, value, timestamp, cid):
        string = "[SENSOR] {0} [{1}/{2}] ({3}) @ {4} <- {5}".format(
            id_, protocol, model, dataType, timestamp, value)
        print(string)
        url = "/rest/items/td_sensor_{0}_{1}_{2}/state".format(protocol, id_, dataType)
        try:
            conn = httplib.HTTPConnection(openhab)
            conn.request('PUT', url, value, headers)
        except:
            print(connErr)
    
    try:
        import asyncio
        loop = asyncio.get_event_loop()
        dispatcher = td.AsyncioCallbackDispatcher(loop)
    except ImportError:
        loop = None
        dispatcher = td.QueuedCallbackDispatcher()

    core = td.TelldusCore(callback_dispatcher=dispatcher)
    callbacks = []
    
    callbacks.append(core.register_device_event(device_event))
    callbacks.append(core.register_raw_device_event(raw_event))
    callbacks.append(core.register_sensor_event(sensor_event))
    
    try:
        while True:
            core.callback_dispatcher.process_pending_callbacks()
            time.sleep(0.5)
    except KeyboardInterrupt:
        pass
```
