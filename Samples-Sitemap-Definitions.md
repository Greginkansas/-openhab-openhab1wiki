Samples for Sitemap definitions

###How to define a setpoint widget for a normal number item

    Setpoint item=Temperature_Setpoint label="Setpoint [%.1f °C]" step=0.5 minValue=15 maxValue=30

###How to refresh an image periodically (every second)

    Image url="http://192.168.0.1/jpg/image.jpg" refresh=1000

###arranging items:

You may find descriptions like:

```    
    Frame label="Demo" {
         Text label="Group Demo" icon="1stfloor" {
                Switch item=Lights mappings=[OFF="All Off"]
                Group item=Heating
                Group item=Windows
                Text item=Temperature
         }
         Text label="Multimedia" icon="video" {
                Selection item=Radio_Station mappings=[0=off, 1=HR3, 2=SWR3, 3=FFH, 4=Charivari]
                Slider item=Volume
            }
    }
```
- This means that you want a frame with a visual label "Demo". Then, inside the frame you want two elements:
- An item called Group Demo with 1stfloor icon that contains 4 items.
- The first one is the group Lights, that has a mapping. It means that when it receives a value of OFF, it might show a "All off" text.
- The second one will be the Heating group.
- etc.
- An item called Multimedia with icon video. It has two elements:
- The Radio_station item that has several mappings
- The Volume item, shown as an Slider.

For more info about other options have a look at the demo.sitemap file.

###How to make a link

First define the link somewhere in an items file:

```
String MY_LINK "<a href='http://link.example/'>Here is a link!</a>"
```

then add it in your sitemap:

```
Text item=MY_LINK
```
