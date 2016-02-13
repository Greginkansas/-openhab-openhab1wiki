## Introduction

The REST API of openHAB serves different purposes. It can be used to integrate openHAB with other systems as it allows read access to items and item states as well as status updates or the sending of commands for items. Furthermore, it gives access to the sitemaps, so that it is the interface to be used by remote user interfaces (e.g. fat clients or fully Javascript based web clients).

The REST interface is very fast, so it is good for real time interaction with openHAB, especially from 3rd party user interfaces such as iRule, openremote or Home Remote (iOS app with apple watch and voice interface).

The REST API furthermore supports server-push, so you can subscribe yourself on change notification for certain resources. Find more about the server-push features in the "Server-Push" section below.

The URIs of the REST API support different media types. This means that you can define the format of the response with an additional HTTP header like 

    Accept: application/xml
For each url, you will find the list of supported media types documented below.

If your client cannot correctly set the HTTP accept header, you also have the choice to append the query "?type=xml", "?type=json" or "?type=jsonp" to the URI to request a specific media type for the response.

Each resource that accepts `application/x-javascript` as a media type returns JSONP results, which can be parameterized by appending `?jsoncallback=callback` to the URI, e.g.

```
    http://localhost:8080/rest/items/Temperature_FF_Office?jsoncallback=callback
```
which yields the following response
```
callback({"type":"NumberItem","name":"Temperature_FF_Office","state":"Undefined","link":"http://localhost:8080/rest/items/Temperature_FF_Office"})
```

The parameter `jsoncallback` is optional. If not provided, `callback` will be used as a default.

## Code Samples

See the following for code samples:
https://github.com/openhab/openhab/wiki/Samples-REST

## Logging In

There are times where the security settings of openHAB requires entry of a username and password but the program making the REST call does not directly support a username and password. This can be handled by supplying the username and password as part of the url using the following pattern:

```
    http://username:password@host:port/rest
```

This also works with my.openhab.

If your username or password contains non-alpha numeric characters you may have to escape those characters. For example, my.openhab uses your email address as the username so the "@" and "." may need to be escaped. See the following for a reference showing the escape codes for all characters:
http://www.w3schools.com/tags/ref_urlencode.asp

In the case of an email, the "@" should be replaced with "%40" and the "." replaced with "%2E".

## Details

The entry url for the REST API is the following:

```
    http://localhost:8080/rest
```
Media types: application/xml, application/json, application/x-javascript

As a response, this will return links to the available resource collections:
```
    <openhab>
        <link type="items">http://localhost:8080/rest/items</link>
        <link type="sitemaps">http://localhost:8080/rest/sitemaps</link>
    </openhab>
```
## Item Resources

The request

```
    http://localhost:8080/rest/items
```
Media types: application/xml, application/json, application/x-javascript

returns a collection of all declared items like
```
    <items>
        <item>
            <type>GroupItem</type>
            <name>Temperature</name>
            <state>20.145533363073167</state>
            <link>http://localhost:8080/rest/items/Temperature</link>
        </item>
        <item>
            <type>NumberItem</type>
            <name>NoOfPairedDevices</name>
            <state>1</state>
            <link>http://localhost:8080/rest/items/NoOfPairedDevices</link>
        </item>
        <item>
            <type>GroupItem</type>
            <name>gFF</name>
            <state>Undefined</state>
            <link>http://localhost:8080/rest/items/gFF</link>
        </item>
        ...
    </items>
```

Single items can hence be accessed like

```
    http://localhost:8080/rest/items/Temperature_FF_Office
```
Media types: application/xml, application/json, application/x-javascript

To directly access an item state, you can access

```
    http://localhost:8080/rest/items/Temperature_FF_Office/state
```
which returns the state as a plain string.

Likewise, you can send a status update using the HTTP verb PUT to the same uri, passing the new state as a plain string argument in the body (encoding text/plain).

In order to send a command to an item, you would use the item uri (`http://localhost:8080/rest/items/Temperature_FF_Office`) and send an HTTP POST with the according command in the request body as text/plain.

It's also possible to send a status update using a HTTP GET request (`http://localhost:8080/CMD?Temperature_FF_Office=12.3`). This way it's actually possible to send status updates simply through a web browser address bar.

Note: Openhab only supports status updates for the current timestamp. You can record "The value is 5, now". You cannot record "The value was 5, at 6am yesterday" to the API as yet.

## Sitemap Resource

The request

```
    http://localhost:8080/rest/sitemaps
```
Media types: application/xml, application/json, application/x-javascript

returns a collection of all declared sitemaps like
```
    <sitemaps>
        <sitemap>
            <name>default</name>
            <link>http://localhost:8080/rest/sitemaps/default</link>
        </sitemap>
        <sitemap>
            <name>demo</name>
            <link>http://localhost:8080/rest/sitemaps/demo</link>
        </sitemap>
    </sitemaps>
```
The content of a single sitemap is available at

The request
```
    http://localhost:8080/rest/sitemaps/demo
```
Media types: application/xml, application/json, application/x-javascript

which returns a result like
```
    <sitemap>
      <name>demo</name>
      <link>http://localhost:8080/rest/sitemaps/demo</link>
      <homepage>
        <id>demo</id>
        <link>http://localhost:8080/rest/sitemaps/demo/demo</link>
        <widget>
          <type>Frame</type>
          <label />
          <icon>frame</icon>
          <widget>
            <type>Group</type>
            <label>First Floor</label>
            <icon>1stfloor</icon>
            <item>
              <type>GroupItem</type>
              <name>gFF</name>
              <state>Undefined</state>
              <link>http://localhost:8080/rest/items/gFF</link>
            </item>
            <linkedPage>
              <id>0000</id>
              <link>http://localhost:8080/rest/sitemaps/demo/0000</link>
              <widget>
                <type>Group</type>
                <label>Bathroom</label>
                <icon>bath</icon>
                <item>
                  <type>GroupItem</type>
                  <name>FF_Bath</name>
                  <state>Undefined</state>
                  <link>http://localhost:8080/rest/items/FF_Bath</link>
                </item>
                [...]
              </widget>
            </linkedPage>
          </widget>
        </widget>
      </homepage>
    </sitemap>
```
You can see that the sitemap information not only contains the static information that the user has provided in the sitemap file, but that it also holds derived data like icons and labels and dynamic group contents, where the groups structure is not explicitly defined in the sitemap.

Single pages can be accessed by adding a page id (as given in the sitemap response) - NOTE: this is the _page id_, not the group name or name entry. In the example above the page id is 0000:

```
    http://localhost:8080/rest/sitemaps/demo/FF_Bath
```
Media types: application/xml, application/json, application/x-javascript

which returns the widgets contained in this page - this happens to works as there is also a page id FF_Bath. In practice it depends on how your sitemap is defined, in many cases there is a sitemap page that corresponds to a group name, but we are getting the sitemap page id, not the group.:
```
    <page>
      <id>FF_Bath</id>
      <link>http://localhost:8080/rest/sitemaps/demo/FF_Bath</link>
      <widget>
        <type>Switch</type>
        <label>Ceiling</label>
        <icon>switch</icon>
        <item>
          <type>SwitchItem</type>
          <name>Light_FF_Bath_Ceiling</name>
          <state>ON</state>
          <link>http://localhost:8080/rest/items/Light_FF_Bath_Ceiling</link>
        </item>
      </widget>
      [...]
      <widget>
        <type>Text</type>
        <label>Bath [undefined]</label>
        <icon>contact</icon>
          <item>
          <type>ContactItem</type>
          <name>Window_FF_Bath</name>
          <state>CLOSED</state>
          <link>http://localhost:8080/rest/items/Window_FF_Bath</link>
        </item>
      </widget>
    </page>
```

## Server-Push

openHAB makes use of the Atmosphere framework for server-push functionality. 
By this, you automatically have the option to use long-polling, HTTP streaming or websockets, depending on what your client supports.  

You can subscribe on the following resources:
- a single item
- all items (rest/items)
- an item group
- an item state
- a single page of a sitemap

Whilst you can subscribe to these things, you do not get notifications on most of them. For example, when using streaming on items (/rest/items/item_name), you can only get responses from a single _item_ (nothing else works, and I've tried), or sitemap pages (/rest/sitemaps/name/page_id). You do not get notifications from all items, item groups, or item states.

The behavior with sitemap pages is that you will only get notification on _items_ that change, in the form of a widget response. This means that if you subscribe to a sitemap page that does not have any items defined (such as a group of groups, or a frame containing only groups) you will not receive any notifications.

Whenever the state of an item that is part of the resource changes, the resource will be returned just as on a regular request (i.e. exactly the same syntax). This is especially useful for sitemap pages as you can instantly refresh your UI whenever some state changes on the currently shown page.

To tell the openHAB server that you want to receive notifications (i.e. use server-push), you have to add the following header to your HTTP request:

```    X-Atmosphere-Transport: websocket|long-polling|streaming```

Moreover it's recommend to set a unique tracking Id for each client (but read the **NOTE**). 

```    X-Atmosphere-tracking-id: unique id```

With the aid the of the tracking id openHAB is able to reduce the network load. openHAB will detect if the actual message is equal to the previous one and will suppress double broadcasts (no I don't know what this means, but I'm leaving it in). The tracking header is also necessary if you want to receive page-label and page-icon updates on streaming connections (see below).

**NOTE:** In practice this is much trickier than it sounds. You can subscribe to any sitemaps page that you want (as explained previously, items only works for _specific items_), but many sitemaps pages do not produce widget responses, in which case you will _only_ receive page-label and icon updates. This is a json example of a page update, on a subscription to a sitemaps page called "Lights", with tracking id enabled:
```
{"widget":{"widgetId":"Lights_18","type":"Slider","label":"Landing [44%]","icon":"slider-40","switchSupport":"true","sendFrequency":"0","item":{"type":"DimmerItem","name":"landingMain","state":"44","link":"http://192.168.100.119:80/rest/items/landingMain"}}}


{"id":"Lights","title":"Lights","icon":"group","link":"http://192.168.100.119:80/rest/sitemaps/nicks/Lights","leaf":"false"}
```

Without the tracking id, you just get the "widget" entry. The tracking id can be any string (not just numbers), but "0" is a special case, it means "send me back a tracking id to use in the future", so if you use "0" as your tracking id, the first response will be a new tracking id. I would just use "1234" or some other random number.

You can safely leave out the tracking id header entry altogether, streaming works fine without it.

In addition, you _must_ specify the data type that you wish to receive, either json or xml (you _must_ include this in the headers - http://xxxxxxx?type=json/xml does not work):

```    Accept: application/json```

If you don't include this, you'll see messages like:

```    15:38:26.773 WARN  o.a.cpr.DefaultBroadcaster[:533] - This message Entry{message=Temperature_Heru_Outdoor (Type=NumberItem, State=6.9), multipleAtmoResources=null, future=org.atmosphere.cpr.BroadcasterFuture@52b0a53d} will be lost```

In the log. This means you haven't defined what kind of response is expected.

**NOTE:** With websockets, you can't include headers in the normal way (the Java API doesn't accept normal headers), you have to attach them as a querystring, so the url would look like:

`    ws://192.168.100.119:8009/rest/sitemaps/nicks/Lights?Accept=application/json`

or

`    http://192.168.100.119:8009/rest/sitemaps/nicks/Lights?Accept=application/json`

Depending on what client you are using. I have this working without sending the X-Atmosphere-Transport: websocket header using python websockets-client.

While registering for changes to _all items_ you will receive every item update which occurs in the openHAB bus. Each response will look like a response from /rest/items/<Item-Name>. This method is designed for small devices, which does not have enough resources to create a websocket connection to every single item separately. _NOTE: This does not seem to work as you can only subscribe to specific items, not groups._

When streaming a connection, openHAB automatically times out the connection after 5 minutes, so you have to reconnect to continue to receive notifications.

Please note: for sitemap resources we have different types of answers.
This depends on the connection type.

### Polling Connections

"X-Atmosphere-Transport: long-polling" should be set as HTTP Header.

long-polling connections will always receive a complete page object. The answer is equal to normal REST api invocations as described above.

This means that if you subscribe to a sitemap page, if anything changes on the page, you receive the entire page, and it is up to you to figure out what changed on it.

### Streaming Connections

"X-Atmosphere-Transport: streaming" or
"X-Atmosphere-Transport: websocket"
should be set as HTTP Header.
Also for websockets, in addition you need to include the normal HTTP websocket upgrade request headers. Most clients take care of this for you if you specify a websockets connection, but see the **NOTE** above for the data type requirement.

HTTP streaming or websocket connections receive only updated objects from the openHAB server. The received update could either be a widget or a page object. If the page label or icon is not changed you will only receive a widget object, but if a label or icon is changed on the page you will additionally receive a page object. 
(For this feature the "X-Atmosphere-tracking-id" header is required).

a widget response looks like
```
    <widgets>
      <widget>
        <widgetId>FF_Bath_0</widgetId>
        <type>Switch</type>
        <label>Ceiling</label>
        <icon>switch-on</icon>
        <item>
          <type>SwitchItem</type>
          <name>Light_FF_Bath_Ceiling</name>
          <state>ON</state>
          <link>http://localhost:8080/rest/items/Light_FF_Bath_Ceiling</link>
        </item>
      </widget>
    </widgets>
```
in case the page label or icon changes you will also receive something like this:
```
    <page>
      <id>030001</id>
      <title>No. of Active Heatings [(1)]</title>
      <icon>heating-on</icon>
      <link>http://localhost:8080/rest/sitemaps/demo/030001</link>
      <parent>
        <id>0300</id>
        <title>Group Demo</title>
        <icon>1stfloor</icon>
        <link>http://localhost:8080/rest/sitemaps/demo/0300</link>
      </parent>
    </page>
```
That's all! The GreenT UI, the iOS client as well as HABDroid use this mechanism to update the pages.

If you are accessing the REST interface from another client (such as a python script), you can use python requests as shown in the Examples Wiki (Note: the example shows long polling to get updates - I don't recommend this, streaming works better. I will try to update the examples), but this will block on responses, so each subscription should be run in it's own thread.

Besides this, server-push can be very helpful for integration with other systems, if they are interested in state changes from openHAB - there is no need to do polling in such a case.

Checkout the atmosphere pages to learn more about client development: https://github.com/Atmosphere/atmosphere

### Accessing historical data from persistence service

In order to access historical data you need to install [HABmin](https://github.com/cdjackson/HABmin/releases).
Afterwards check how to use its [REST Persistance API in the according documentation](https://github.com/cdjackson/HABmin/wiki/REST-Persistence).