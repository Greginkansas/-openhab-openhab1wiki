Examples for accessing REST API

* [jquery](Samples-REST#jquery)
* [cURL](Samples-REST#curl)
* [PHP](Samples-REST#php)
* [Python](Samples-REST#python)

## Introduction

This page has samples in multiple languages for utilizing the three main REST interfaces for Items in OpenHAB:

* **Send Command** - This will send a command to an item to tell it to take some action (e.g. turn on light).

* **Send Status** - This is to indicate that the status of an item has changed (e.g. window is now open).

* **Get Status** - Gets the current status of an item.  (This can also be used to continuously get updates as shown in the Python section).

There is also a sitemap interface.

## Language Samples

### jquery

Accessing REST API via jquery (tested with jquery 2.0 and Chrome v26.0

_Get state of an item:_

```javascript
function getState()
{
    var request = $.ajax
    ({
        type       : "GET",
        url        : "http://192.168.100.21:8080/rest/items/MyLight/state"
    });

    request.done( function(data) 
    { 
        console.log( "Success: Status=" + data );
    });

    request.fail( function(jqXHR, textStatus ) 
    { 
        console.log( "Failure: " + textStatus );
    });
}
```

_Set state of an item:_
```javascript
function setState( txtNewState )
{
    var request = $.ajax
    ({
        type       : "PUT",
        url        : "http://192.168.100.21:8080/rest/items/MyLight/state",
        data       : txtNewState, 
        headers    : { "Content-Type": "text/plain" }
    });
    
    request.done( function(data) 
    { 
        console.log( "Success" );
    });
    
    request.fail( function(jqXHR, textStatus ) 
    { 
        console.log( "Failure: " + textStatus );
    });
}
```

_Send command to an item:_
```javascript
function sendCommand( txtCommand )
{
    var request = $.ajax
    ({
        type       : "POST",
        url        : "http://192.168.100.21:8080/rest/items/MyLight",
        data       : txtCommand,
        headers    : { 'Content-Type': 'text/plain' }
    });

    request.done( function(data) 
    { 
        console.log( "Success: Status=" + data );
    });

    request.fail( function(jqXHR, textStatus ) 
    { 
        console.log( "Failure: " + textStatus );
    });
}
```
### cURL

Accessing REST API via [cURL](http://curl.haxx.se). cURL is useful on shell scripts (Win/Linux/OS X) or e.g. on Automator (OS X).

_Get state of an item:_
```shell
    curl http://192.168.100.21:8080/rest/items/MyLight/state
```
_Set state of an item:_
```shell
    curl --header "Content-Type: text/plain" --request PUT --data "OFF" http://192.168.100.21:8080/rest/items/MyLight/state
```
_Send command to an item:_
```shell
    curl --header "Content-Type: text/plain" --request POST --data "ON" http://192.168.100.21:8080/rest/items/MyLight
```
### PHP

Accessing REST API via PHP.  Simple PHP function to post a command to a switch using the REST interface.

_Send command to an item:_
```php
    function sendCommand($item, $data) {
      $url = "http://192.168.1.121:8080/rest/items/" . $item;
    
      $options = array(
        'http' => array(
            'header'  => "Content-type: text/plain\r\n",
            'method'  => 'POST',
            'content' => $data  //http_build_query($data),
        ),
      );
    
      $context  = stream_context_create($options);
      $result = file_get_contents($url, false, $context);
    
      return $result;
    }
```
Example function use:

    sendCommand("doorbellSwitch", "ON");

If the post was successful the function will return the state you set, EG above returns "ON"

### Python

Accessing REST API via Python.  Note that the rget status interface is set up to continuously receive updates rather than just getting a one time response.  This is done with the "polling header" and the last section decodes the JSON response.

_Send command to an item_
```python
def post_command(self, key, value):
    """ Post a command to OpenHAB - key is item, value is command """
    url = 'http://%s:%s/rest/items/%s'%(self.openhab_host,
                                self.openhab_port, key)
    req = requests.post(url, data=value,
                            headers=self.basic_header())
    if req.status_code != requests.codes.ok:
        req.raise_for_status()
```
_Set state of an item_
```python
def put_status(self, key, value):
    """ Put a status update to OpenHAB  key is item, value is state """
    url = 'http://%s:%s/rest/items/%s/state'%(self.openhab_host,
                                self.openhab_port, key)
    req = requests.put(url, data=value, headers=self.basic_header())
    if req.status_code != requests.codes.ok:
        req.raise_for_status()     
```
_Get state updates of an item (using long-polling)_
```python
def get_status(self, name):
    """ Request updates for any item in group NAME from OpenHAB.
     Long-polling will not respond until item updates.
    """
    # When an item in Group NAME changes we will get all items in the group 
    # and need to determine which has changed
    url = 'http://%s:%s/rest/items/%s'%(self.openhab_host,
                                    self.openhab_port, name)
    payload = {'type': 'json'}
    try:
        req = requests.get(url, params=payload,
                            headers=self.polling_header())
        if req.status_code != requests.codes.ok:
            req.raise_for_status()
        # Try to parse JSON response
        # At top level, there is type, name, state, link and members array
        members = req.json()["members"]
        for member in members:
            # Each member has a type, name, state and link
            name = member["name"]
            state = member["state"]
            do_publish = True
            # Pub unless we had key before and it hasn't changed
            if name in self.prev_state_dict:
                if self.prev_state_dict[name] == state:
                    do_publish = False
            self.prev_state_dict[name] = state
            if do_publish:
                self.publish(name, state)
```

_HTTP Header definitions_
```python
def polling_header(self):
    """ Header for OpenHAB REST request - polling """
    self.auth = base64.encodestring('%s:%s'
                       %(self.username, self.password)
                       ).replace('\n', '')
    return {
        "Authorization" : "Basic %s" % self.cmd.auth,
        "X-Atmosphere-Transport" : "long-polling",
        "X-Atmosphere-tracking-id" : self.atmos_id,
        "X-Atmosphere-Framework" : "1.0",
        "Accept" : "application/json"}

def basic_header(self):
    """ Header for OpenHAB REST request - standard """
    self.auth = base64.encodestring('%s:%s'
                       %(self.username, self.password)
                       ).replace('\n', '')
    return {
            "Authorization" : "Basic %s" %self.auth,
            "Content-type": "text/plain"}
```

_Get state updates of an item (using streaming) NOTE: this is a class method, not a stand alone example_
```python
    def get_status_stream(self, item):
        """
        Request updates for any item in item from OpenHAB.
        streaming will not respond until item updates. Can also use 
        Sitemap page id (eg /rest/sitemaps/name/0000) as long as it
        contains items (not just groups of groups)
        auto reconnects while parent.connected is true.
        This is meant to be run as a thread
        """
        
        connect = 0     #just keep track of number of disconnects/reconnects
        
        url = 'http://%s:%s%s'%(self.openhab_host,self.openhab_port, item)
        payload = {'type': 'json'}
        while self.parent.connected:
            if connect == 0:
                log.info("Starting streaming connection for %s" % url)
            else:
                log.info("Restarting (#%d) streaming connection after disconnect for %s" % (connect, url))
            try:
                req = requests.get(url, params=payload, timeout=(310.0, 310),   #timeout is (connect timeout, read timeout) note! read timeout is 310 as openhab timeout is 300
                                headers=self.polling_header(), stream=True)
                if req.status_code != requests.codes.ok:
                    log.error("bad status code")
                    req.raise_for_status()
                    
            except requests.exceptions.ReadTimeout, e: #see except UnboundLocalError: below for explanation of this
                if not self.parent.connected:   # if we are not connected - time out and close thread, else retry connection.
                    log.error("Read timeout, exit: %s" % e)
                    break
                    
            except (requests.exceptions.HTTPError, requests.exceptions.ConnectTimeout, requests.exceptions.ConnectionError) as e:
                log.error("Error, exit: %s" % e.message)
                break
            #log.debug("received response headers %s" % req.headers)
            log.info("Data Received, streaming connection for %s" % url)
            connect += 1
            try:
                while self.parent.connected:
                    message = ''
                    content = {}
                    for char in req.iter_content():   #read content 1 character at a time
                        try:
                            if char:
                                #log.debug(char)
                                message += char
                                content = json.loads(message)
                                break
                            
                        except ValueError:      #keep reading until json.loads returns a value
                            pass
                    #log.debug(content)
                    if len(content) == 0:
                        raise requests.exceptions.ConnectTimeout("Streaming connection dropped")
                        
                    members = self.extract_content(content)        
                    self.publish_list(members)
            
            except UnboundLocalError:   #needed because streaming on single item does not time out normally - so thread hangs.
                pass
            except (timeout, requests.exceptions.ConnectTimeout, requests.exceptions.ConnectionError) as e:
                log.info("Socket/Read timeout: %s" % e.message)
            except Exception, e:
                log.error("Stream Unknown Error: %s, %s" % (e, e.message))
                log.error("logging handled exception - see below")
                log.exception(e)
                
        log.info("Disconnected, exiting streaming connection for %s" % url)
        if item in self.streaming_threads:
            del(self.streaming_threads[item])
            log.debug("removed %s from streaming_threads" % item)

    def polling_header(self):
        """ Header for OpenHAB REST request - streaming """
        
        self.auth = base64.encodestring('%s:%s'
                        %(self.username, self.password)
                        ).replace('\n', '')
        return {
            #"Authorization" : "Basic %s" % self.auth,
            "X-Atmosphere-Transport" : "streaming",
            #"X-Atmosphere-tracking-id" : self.atmos_id,
            "Accept" : "application/json"}

    def basic_header(self):
        """ Header for OpenHAB REST request - standard """
        
        self.auth = base64.encodestring('%s:%s'
                        %(self.username, self.password)
                        ).replace('\n', '')
        return {
                #"Authorization" : "Basic %s" %self.auth,
                "Content-type": "text/plain"}
                
    def extract_content(self, content):
        """ extract the "members" or "items" from content, and make a list """
        
        # sitemap items have "id" and "widget" keys. "widget is a list of "item" dicts. no "type" key.
        # items items have a "type" key which is something like "ColorItem", "DimmerItem" and so on, then "name" and "state". they are dicts
        # items groups have a "type" "GroupItem", then "name" and "state" (of the group) "members" is a list of item dicts as above
        
        if "type" in content:                   #items response
            if content["type"] == "GroupItem":
                # At top level (for GroupItem), there is type, name, state, link and members list
                members = content["members"]    #list of member items
            elif content["type"] == "item":
                members = content["item"]       #its a single item dict *not sure this is a thing* 
            else:
                members = content               #its a single item dict
        elif "widget" in content:               #sitemap response
            members = content["widget"]["item"] #widget is a list of items, (could be GroupItems) these are dicts
        elif "item" in content:
            members = content["item"]           #its a single item dict
        else:
            members = content                   #don't know...
        #log.debug(members)
        
        if isinstance(members, dict):   #if it's a dict not a list
            members = [members]         #make it a list (otherwise it's already a list of items...)
            
        return members
```

_Get state updates of an item/page (using websockets) you can decode the data in the same way as the above example_
```python
    #!/usr/bin/env python

    import websocket    #need to run pip install websockets-client to get this module

    class WSClient():
     
        def __init__(self):
            websocket.enableTrace(False) #True for debugging
           
        def connect(self, url):
            print("connecting to %s" % url)
            
            self.ws = websocket.WebSocketApp(url,
            on_message = self.on_message,
            on_error = self.on_error,
            on_close = self.on_close)
            self.ws.on_open = self.on_open
            self.ws.run_forever()
     
        def on_message(self, ws, message):
            print("Received:")
            print(message)
     
        def on_error(self, ws, error):
            print(error)
     
        def on_close(self, ws):
            print("connection closed")
     
        def on_open(self, ws):
            print("connected")
            
        def close(self, ws):
            ws.close()
     
    if __name__ == "__main__":
        try:
            client = WSClient()
            client.connect("ws://192.168.100.119:8009/rest/sitemaps/nicks/Lights?Accept=application/json")
            #client.connect("ws://192.168.100.119:8009/rest/items/landingMain?Accept=application/json")
            #client.connect("ws://192.168.100.119:8009/rest/items/sensortag1IRAmbientTemp?Accept=application/json")
           
        except (KeyboardInterrupt, SystemExit):
            print("System exit Received - Exiting program")
            client.close()
            print(" **** Program Ended ****")

```