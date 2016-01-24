# Use Google MAP V3 API
 * [Introduction](GoogleMap#Introduction)
 * [Pre-Requisits](GoogleMap#pre-requisits)
 * [Configuration](GoogleMap#configuration)
  * [Parse the raw data ...](GoogleMap#parse-the-raw-data-)
  * [The HTML code ...](GoogleMap#the-html-code-)

## Introduction

After the setup of [MQTT](MQTT-Binding) and [MQTTitude](Mqttitude-Binding) bindings to provide geo-fencing I decided some more color in the UI would be nice. The idea was to show a map that centers on my home and automatically scales to show the location of the inhabitants:

![](https://dl.dropboxusercontent.com/u/1781347/wiki/2015-06-11_15_51_06.png)

For people familiar with Google API and Ajax that is probably nothing fancy ... but for those (like me) that are new to that topic my setup might give some help and a head start.

## Pre-Requisits

For this example to work you'll need a proper [MQTT](MQTT-Binding) and [OwnTracks](owntracks.org) setup. 
Once you have the "raw" data available in OH you're ready ...
###Items...
```Xtend
Location locationPatrik
String	 mqttPositionPatrikRaw  "Patrik Raw Data"	{ mqtt="<[home:owntracks/Lex/LexLuther:state:default]" }
String   mqttPatrikLatitude     "Patrik's Lat"
String   mqttPatrikLongitude    "Patrik's Lon"
String   mqttPatrikAccuracy     "Patrik's Accuracy"
String   mqttHtcOneBattery      "Patrik's HTC One Battery [%s%%]"		<battery>	(Phone, MQTT, Battery)
```

## Configuration
### Parse the raw data ...
For each person you'ld like to show on the map you'll need to have latitude and longitude data available. A rule can be used to parse the data received:

```Xtend
import org.openhab.core.library.types.*

rule "MqttPostionParsePatrik"
  when 
    Item mqttPositionPatrikRaw changed
  then
    val String json = (mqttPositionPatrikRaw.state as StringType).toString
	// {"_type": "location", "lat": "47.5010314", "lon": "8.3444293",
	//    "tst": "1422616466", "acc": "21.05", "batt": "40"}
	val String type = transform("JSONPATH", "$._type", json)
	if (type == "location") {
	  val String lat  = transform("JSONPATH", "$.lat", json)
	  val String lon  = transform("JSONPATH", "$.lon", json)
	  val String acc  = transform("JSONPATH", "$.acc", json)
	  val String batt = transform("JSONPATH", "$.batt", json)

      mqttPatrikLatitude.postUpdate(lat)
      mqttPatrikLongitude.postUpdate(lon)
      locationPatrik.postUpdate(new PointType(lat + "," + lon))
      mqttPatikAccuracy.postUpdate(new DecimalType(acc))
      mqttHtcOneBattery.postUpdate(new PercentType(batt))
	}
  end
```

### The HTML code ...
The following code will display a map based on your home location; and auto zoom to show all markers:

```html
<!DOCTYPE html>
<html>
  <head>    
    <style type="text/css"> 
    <!--
    .Flexible-container {
      position: relative;
      padding-bottom: 0px;
      padding-top   : 0px;
      height        : 345px ;
      overflow: hidden;
    }

    .Flexible-container iframe,   
    .Flexible-container object,  
    .Flexible-container embed {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }
   -->
   </style>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.0/jquery.min.js"></script>
    <script type="text/javascript" src="https://maps.googleapis.com/maps/api/js?v=3.exp&libraries=places,drawing,geometry"></script>

    <script type="text/javascript">
        ////////////////////////////////////////////////////////////////////////
        // Google Maps JavaScript API:
        // https://developers.google.com/maps/documentation/javascript/?hl=de
        // Marker Icons:
        // https://sites.google.com/site/gmapsdevelopment/
        ////////////////////////////////////////////////////////////////////////

        ////////////////////////////////////////////////////////////////////////
        // JQuery
        ////////////////////////////////////////////////////////////////////////
        
        var map = null;
        //make an empty bounds variable
        var bounds = new google.maps.LatLngBounds();

        // LatLng's we want to show 
        var latlngHome   = new google.maps.LatLng("47.501006", "8.344842");
        var latlngPatrik = new google.maps.LatLng("47.501006", "8.344842"); // initialize to home ...
        var latlngKarin  = new google.maps.LatLng("47.501006", "8.344842"); // initialize to home ...
        
        var map_options = { center    : latlngHome,
                            zoom      : 10,
                            mapTypeId : google.maps.MapTypeId.ROADMAP };
         
        $( "#map_canvas" ).ready($(function() {
          var map_canvas = document.getElementById('map_canvas');
          map = new google.maps.Map(map_canvas, map_options)
          
          var marker = new google.maps.Marker({
                            position  : latlngHome,
                            map       : map,
                            icon      : 'http://maps.google.com/mapfiles/kml/pal2/icon10.png',
                            title     : "Ehrendingen"
                        })
                        
           bounds.extend(latlngHome);
        }))

        $( document ).ready($(function() {
            // ******************************************************************************
            $.ajax({
              url     : "../rest/items/locationPatrik/state/",
              data    : {},
              success : function( data ) {
                  if ( map == null) { return; }
                  if ( data == "Uninitialized") { return; }
                  
                  var coords = data.split(',');
                  var latlngPatrik = new google.maps.LatLng(coords[0],coords[1]);
                  
                  var marker = new google.maps.Marker({
                    position  : latlngPatrik,
                    map       : map,
                    icon      : 'http://maps.google.com/mapfiles/ms/icons/green-dot.png',
                    title     : "Patrik"
                  }) // end of [Marker]
                  
                  $.ajax({
                    url     : "../rest/items/mqttPatikAccuracy/state/",
                    data    : {},
                    success : function( data ) {
                    if ( data == "Uninitialized") { return; }
                      var accuracy = parseInt(data);
                      var circle = new google.maps.Circle({
                        center        : latlngPatrik,
                        radius        : accuracy,
                        map           : map,
                        strokeColor   : '#00FF00',
                        strokeOpacity : 0.8,
                        strokeWeight  : 2,
                        fillColor     : '#00FF00',
                        fillOpacity   : 0.35,
                      }); // end of [Circle]
                      
                      bounds.extend(latlngPatrik);
                      map.fitBounds(bounds);
                      
                    } // end of [function]
                  }) // end of [$.ajax]
                } // end of [function]
            }) // end of [$.ajax]
            // ******************************************************************************
            $.ajax({
              url     : "../rest/items/locationKarin/state/",
              data    : {},
              success : function( data ) {
                  if ( map == null) { return; }
                  if ( data == "Uninitialized") { return; }
                  
                  var coords = data.split(',');
                  var latlngKarin = new google.maps.LatLng(coords[0],coords[1]);
                  
                  var marker = new google.maps.Marker({
                    position  : latlngKarin,
                    map       : map,
                    icon      : 'http://maps.google.com/mapfiles/ms/icons/blue-dot.png',
                    title     : "Karin"
                  }) // end of [Marker]
                  
                  $.ajax({
                    url     : "../rest/items/mqttKarinAccuracy/state/",
                    data    : {},
                    success : function( data ) {
                    if ( data == "Uninitialized") { return; }
                      var accuracy = parseInt(data);
                      var circle = new google.maps.Circle({
                        center        : latlngKarin,
                        radius        : accuracy,
                        map           : map,
                        strokeColor   : '#00FF00',
                        strokeOpacity : 0.8,
                        strokeWeight  : 2,
                        fillColor     : '#00FF00',
                        fillOpacity   : 0.35,
                      }); // end of [Circle]
                      
                      bounds.extend(latlngKarin);
                      map.fitBounds(bounds);
                    } // end of [function]
                  }) // end of [$.ajax]
                } // end of [function]
            }) // end of [$.ajax]
            // ******************************************************************************
        }))
    </script>
  </head>
  <body>
    <div id="map_canvas" class="Flexible-container" />
  </body>
</html>
```

Store that file on your OH system in "\webapps\static" (e.g. as Map.html). You can then use it in our site definitions; or directly in a browser.