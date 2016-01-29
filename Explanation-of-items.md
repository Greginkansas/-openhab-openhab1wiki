* [Introduction](#introduction)
* [Syntax](#syntax)
	* [Real life example](#real-life-example)
	* [Item type](#item-type)
		* [Group](#group)
	* [Item name](#item-name)
	* [Label text](#label-text)
		* [Formatting](#formatting)
		* [Transforming](#transforming)
	* [Icon name](#icon-name)
		* [Dynamic icons](#dynamic-icons)
	* [Groups](#groups)
	* [Binding config](#binding-config)
* [Further examples](Explanation-of-items/#further-examples)

# Introduction
* Items are objects that can be read from or written to in order to interact with them.
* Items can be bound to bindings i.e. for reading the status from e.g. KNX or for updating them. Read the wiki page for the respective binding for more help and examples.
* Items can be defined in files in folder ```${openhab_home}/configurations/items```.
* All item definition files have to have the file extension `.items`. Just create a new file called thenameyouwish.items.
* Groups are also defined in the .items files. Groups can be inside groups, and items can be in none, one or more groups. 

Typically items are defined using the openHAB Designer by editing the items definition files. Doing so you will have full IDE support like syntax checking, context assist etc.

# Syntax
Items are defined in the following syntax:

    itemtype itemname ["labeltext"] [<iconname>] [(group1, group2, ...)] [{bindingconfig}]

Parts in square brackets ([]) are optional.

## Real life example
    Number Temperature_GF_Living "Temperature [%.1f °C]" <temperature> (GF_Living) {knx="1/0/15+0/0/15"}

Above example defines an item 
* of [type](#item-type) `Number`
* with [name](#item-name) `Temperature_GF_Living`
* formatting its [output](#label-text) in format `xx.y °C`
* displaying [icon](#icon-name) `temperature`
* belonging to [group](#groups) `GF_Living`
* bound to openHAB [binding](#binding-config) `knx` with write group address `1/0/15` and listening group address `0/0/15`

## Item type
The item type defines which kind of values can be stored in that item and which commands can be send to it. Each item type has been optimized for certain components in your smart home. This optimization is reflected in the data types and commands types. Little example for better understanding: Have a look at the following table and find the itemtype "Color". A Philips Hue RGB light bulb provides you three information. Is the bulb on or off, its current brightness and the color. If you want to change one of these values you can use four commands. Switching the bulb on or off, increasing or decreasing the brightness, setting the brightness directly to a specific value and you can change the bulb's color.

OpenHAB 1.8 provides you the following item types (alphabetical order):

Item Type | Description | Accepted Data Types | Accepted Command Types
----------|-------------|---------------------|-----------------------
Call | This type can be used for items that are dealing with telephony functionality. | Call | -
Color | Can be used for color values, e.g. for LED lights | OnOff, Percent, HSB | OnOff, IncreaseDecrease, Percent, HSB
Contact | Can be used for sensors that return an "open" or "close" as a state. This is useful for doors, windows, etc. | OpenClosed | -
DateTime | Stores a timestamp including a valid time zone. | DateTime | DateTime
Dimmer | Accepts percent values to set the dimmed state. Can also be used as a switch by accepting ON/OFF commands (though this only mimics a Switch by sending 0% and 100% for ON/OFF)  | Percent | OnOff, IncreaseDecrease, Percent
Group | Item to nest other items / collect them in groups | - | -
Location | Can be used to store GPS related informations, addresses, etc. by latitude, longitude and altitude | Point | Point
Number | Has a decimal value and is usually used for all kinds of sensors, like temperature, brightness, wind, etc. It can also be used as a counter or as any other thing that can be expressed as a number. | Decimal | Decimal
Rollershutter | Allows the control of roller shutters, i.e. moving them up, down, stopping or setting it to close to a certain percentage. | UpDown, Percent | UpDown, StopMove, Percent
String | Can be used for any kind of string or textuell representation of a DateTime. | String, DateTime | String
Switch | Represents a normal switch that can be ON or OFF. Useful for normal lights, presence detection, etc. | OnOff | OnOff

### Group
The item type _group_ is used to define a group in which you can nest/collect other items, including other groups. You don't need groups, but they are a great help for your openHAB configuration. Groups are supported in sitemaps, rules, functions, the openhab.cfg and more places. In all these places you can either write every single item, for example your 6 temperature sensors, or you just put all into one group and use the group instead. A typical and minimal group definition is:

    Group TemperatureSensors

Group items can also be used to easily determine one or more items with a defined value or can be used to calculate a value depending on all values within the group. Please note that this can only be used if all items in the group have the same type. You have to add this type of the items and the desired function to the item type:

    Group:itemtype:function itemname ["labeltext"] [<iconname>] [(group1, group2, ...)]

Function | Description
---------|------------
AND(value1, value2) | This does a logical 'and' operation. Only if all items are of 'value1' this is returned, otherwise the 'value2' is returned.
AVG | Calculates the numeric average over all item values of decimal type.
MAX | This calculates the maximum value of all item values of decimal type.
MIN | This calculates the minimum value of all item values of decimal type.
NAND(value1, value2) | This does a logical 'nand' operation. The value is 'calculated' by the normal 'and' operation and than negated by returning the opposite value. E.g. when the 'and' operation calculates the value1 the value2 will be returned and vice versa. 
NOR(value1, value2) | This does a logical 'nor' operation. The value is 'calculated' by the normal 'or' operation and than negated by returning the opposite value. E.g. when the 'or' operation calculates the value1 the value2 will be returned and vice versa. 
OR(value1, value2) | Does a logical 'or' operation. If at least one item is of 'value1' this is returned, otherwise the 'value2' is returned.
SUM | Calculates the sum of all items in the group.

## Item name

The item name is the unique name of the item which is used e.g. in the sitemap definition or rule definition files to access the specific item. The name must be unique across all item files. The name should only consist of letters, numbers and the underscore character. Spaces cannot be used.

## Label text

The label text is used on the one hand side to display a description for the specific item e.g. in the sitemap, on the other hand to format or transform the output of the item. If you want to display a special character you have to mask it with it a '%'. So to display one '%' write '%%'.

### Formatting

Formatting is done applying [Java formatter class syntax](http://docs.oracle.com/javase/7/docs/api/java/util/Formatter.html#syntax), therefore the syntax is

    %[argument_index$][flags][width][.precision]conversion

Only the leading '%' and the trailing 'conversion' are mandatory. The **argument_index$** must be used if you want to convert the value of the item several times within the label text or if the item has more than one value. Look at the DateTime and Call item in the following example.

    Number    MyTemperature  "Temperature [%.1f] °C"          { someBinding:somevalue }
    String    MyString       "Value: [%s]"                    { someBinding:somevalue }
    DateTime  MyLastUpdate   "Last Update: [%1$ta %1$tR]"     { someBinding:somevalue }
    Call      IncomingCall   "Incoming call: [%1$s to %2$s]"  { someBinding:somevalue }

The output would look like this:

    Temperature 23.2 °C
    Value: Lorem ipsum
    Last Update: Sun 15:26
    Incoming call: +1555555555 to +49555555555

### Transforming

Another possibility in label texts is to use a transformation. They are used for example to translate a status into another language or convert technical value into human readable ones. To do this you have to create a .map file in your ```${openhab_home}/configurations/transform``` folder. As you have a look at that directory there are still some .map files. These files are typical key/value pair files.

    key1=value1
    key2=value2
    ...

Let's make a small example to illustrate this function. If you have a sensor which returns you the number 0 for a closed window and 1 for an open window, you can transform these values into the words "open" or "closed". Create a map file named window.map for example and add the desired keys and values.

    0=closed
    1=opened
    UNDEFINED=unknown
    -=unknown

Next we define two items. One showing the raw value as it is provided from our sensor and one with transformed value.

    Number WindowRaw          "Window is [%d]"                  { someBinding:somevalue }
    Number WindowTransformed  "Window is [MAP(window.map):%s]"  { someBinding:somevalue }

The output will be

    Window is 1
    Window is opened

## Icon name

The icon name is used to reference a png image file from folder ```${openhab_home}/webapps/images/```. These icons are used in the openHAB frontends. OpenHAB provides you a set of basic icons. To use on of the images just write the file name without it's ending (".png") between a '<' and '>'.

Feel free to put your own icons into that directory. The images must be in png format, having a size of 32x32 pixel and a name with only small letters and the underscore. There was a thread in the community board where some icon source was provide:

* [[http://www.intranet-of-things.com/software/downloads]]
* [[https://github.com/OpenAutomationProject/knx-uf-iconset]]

### Dynamic icons
You can dynamically change the icon depending on the item state. You have to provide one icon file per state with the states name append to the icons name.

    present.png
    present-off.png
    present-on.png

If you want to use the dynamically items just use the image name without the added states.

    Switch  MeAtHome  "I'm at home!"  <present>  { somebinding:someconfig }

A file among files having such additions that has no addition represents an uninitialized state.

## Groups
Items can be linked to specific groups by referencing these in a comma separated list embraced by round brackets. An item defined like

    Number  MyTemperature  (Group_TemperaturesInside, Group_Temperatures)

would be member of the groups ```Group_TemperaturesInside``` and ```Group_Temperatures```.

## Binding config
The binding config is the most import part of an item. It defines from where the item gets it values and where a given value/command should be send. Bind an item to a binding by adding a binding definition in curly brackets at the end of the item definition

    { ns="bindingconfig" }

Where _ns_ is the namespace for a certain binding like "knx", "bluetooth", "serial". Every binding defines what values must be given in the bindingconfig string. That can be the id of a sensor, an ip or mac address or anything else. You must have a look at your [[Bindings]] configuration section, to know what to use. Some typical examples are:

    Switch  Light_Floor        "Light at Floor"                { knx="1/0/15+0/0/15" }
    Switch  Presence           "I'm at home"                   { bluetooth="123456ABCD" }
    Switch  Doorbell           "Doorbell"                      { serial="/dev/usb/ttyUSB0" }
    Contact Garage             "Garage is [MAP(en.map):%s]"    { zwave="21:command=sensor_binary,respond_to_basic=true" }
    String  Error_Ventilation  "Error in Ventilation %s"       { comfoair="error_message" }
    Number  DiningRoomTemp     "Maximum Away Temp. [%.1f °F]"  { nest="<[thermostats(Dining Room).away_temperature_high_f]" }

# Further examples
1. Further examples for defining items can be found in our [openHAB-samples](https://github.com/openhab/openhab/wiki/Samples-Item-Definitions) section.
1. The openHAB runtime comes with a [demo items file](https://github.com/openhab/openhab-distro/blob/master/features/openhab-demo-resources/src/main/resources/items/demo.items).
1. Every [binding](bindings) provides it own item samples for better understanding the usage of the binding.