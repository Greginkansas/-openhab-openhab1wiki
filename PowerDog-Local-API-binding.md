Documentation of the PowerDog Local API Binding Bundle

## Introduction

This is an OpenHAB binding for an [eco-data PowerDog](http://www.eco-data.de/produkte.html). It supports the [PowerAPI Local Device API](http://api.power-dog.eu/documentation/) in the only available revision [v0.b](http://api.power-dog.eu/documentation/DOCUMENATION/PowerAPI%20Local%20Device%20API%20Description_v0.b.pdf).
The binding allows to query multiple PowerDogs if they are reachable in your network via TCP/IP. For querying, it is expected that your PowerDog is already set up.
The binding supports reading of all PowerDog variables available via PowerDog Local API. It also supports writing of PowerAPI sensors. The binding uses the [Redstone XML-RPC Library](http://xmlrpc.sourceforge.net/) which can be downloaded [here from Sourceforge](http://sourceforge.net/projects/xmlrpc/) for accessing the PowerAPI.

**_This binding will be available from 1.9.0 onwards or is available from corresponding snapshot_**

**_The binding was tested successfully using the OpenHAB 2.1 release_**


## Installation

The current OpenHAB distributions (1.x or 2.x) do not include the redstone RPC library needed. You can find it e.g. [here](https://github.com/wuellueb/openhab/blob/powerdoglocalapi_based_on_current_OH_master/bundles/binding/org.openhab.binding.powerdoglocalapi/lib/xmlrpc-client-1.1.1.jar) - please copy it to the same (addons-) folder as the org.openhab.binding.powerdoglocalapi-1.x.x.jar.


## Binding Configuration

You will normally need some basic configuration for accessing a PowerDog. The following settings can be made for one or several PowerDogs in openhab.cfg. If you are using OpenHAB 2.x, please copy the following sections to a file powerdoglocalapi.cfg and remove the prefix `powerdoglocalapi:` of all parameters.

    ############################### eco-data PowerDog LocalAPI Binding #####################
    #
    # Refresh Interval in ms, determines the fastest refresh rate for any PowerDog
    # (optional, defaults to 300000ms = 5min)
    #powerdoglocalapi:refresh=

The `powerdoglocalapi:refresh` value is the globally fastest refresh rate for all PowerDog servers configured. If you configure lower values for the single PowerDog servers or items, this value will override it. By default the value is set to 5 minutes - be aware that lower values mean more communication, which might result in too much load of your PowerDog.

Using different `serverId`s, you can configure several PowerDog servers at a time.

    # PowerDog IP or host name (optional, defaults to powerdog)
    #powerdoglocalapi:<serverId>.host=

    # PowerDog port for local API via XML-RPC (optional, defaults to 20000)
    #powerdoglocalapi:<serverId>.port=

Normally you will need to set the `host` value or configure your DHCP server accordingly. The `powerdoglocalapi:<serverId>.host` value is the host name to access the PowerDog - it can be an IP adress or a local DNS entry and defaults to `powerdog`. The `powerdoglocalapi:<serverId>.port` value of the PowerDog Local API is by default 20000, but you might need configure this if changed or port redirection is used.

    # Refresh interval in ms used to poll values from the specific PowerDog via local API
    # should be a multiple of powerdoglocalapi:refresh
    # if this value is lower than powerdoglocalapi:refresh the binding will
    # use powerdoglocalapi:refresh instead
    # (optional, defaults to 300000ms = 5min)
    #powerdoglocalapi:<serverId>.refresh=

You can set different refresh rates per PowerDog unit. The `powerdoglocalapi:<serverId>.refresh=` value should be the same or a multiple of `powerdoglocalapi:refresh`.

    # Password set for PowerDog PowerAPI (On a Powerdog set to unlock key by default)
    # defaults to empty string if not set
    #powerdoglocalapi:<serverId>.password=1abcd2

Normally you will need to set this value. The `powerdoglocalapi:<serverId>.password` value defaults for the binding to an empty string, but on a PowerDog it is set to the unlock key by default.


## Item Configuration

In order to bind an generic item to the device, you need to provide PowerDog Local API configuration settings in your item file (in the folder configurations/items) containing at least the value ID of the device you wish to control used on your PowerDog.

The syntax of the binding configuration strings accepted is the following:

    powerdoglocalapi="<<serverId>:<valueID>:<refreshInterval>[:<variableName>]"

The `<` in front of `<serverId>` tells the binding to *read* the following value, while `>` means to write (only applicable for PowerAPI sensors).

The `serverId` corresponds to the device which is introduced in openhab.cfg. 'serverId' can be any alphanumeric string as long as it is the same in the binding and
configuration file. *NOTE*: The parameter is case sensitive!

The `valueID` corresponds to the PowerDog configuration key or value ID of the device you want to query. 
*How to get the PowerDog key or value ID*: Please configure a (even wrong example) item and run OpenHAB in debug mode. In the generated log file you will find the complete response of your PowerDog server indicating all keys/valueIDs.

The `refreshInterval` is the interval in milliseconds to refresh the data. If the value is set to a value lower than the refresh rate of the corresponding PowerDog unit, it will be set to that value. *NOTE*: For values which are written (instead of being read), this is the minimum time between write operations. Values will be only written if updated, so you will need to take care to update them e.g. regularly by using a rule.

The 'variableName' is optional and defaults to `Current_Value`, as this is the variable where PowerDog returns the current value. You might also want to query e.g. for `Last_Read_Average` or even values like the static `Unit` (see the PowerDog PowerAPI Local Device API documentation) and need to add `variableName` in that case.

The `value-name` corresponds to the value you want to query.

Here are some examples for valid binding configuration strings:

    { powerdoglocalapi="<serverId:arithmetic_1234567890:300000" }

These two lines have actually the same result:

    { powerdoglocalapi="<powerdog:pv_global_1234567890:300000" }
    { powerdoglocalapi="<powerdog:pv_global_1234567890:300000:Current_Value" }
    
This is a string result:

    { powerdoglocalapi="<powerdog:impulsecounter_1234567890:300000:Unit_1000000" }
    
This is a configuration for writing a value to a PowerAPI sensor. Configuration of the `variableName` is ignored in that case, because writing to PowerDog Local API only affects the `Current_Value`.

    { powerdoglocalapi=">powerdog:powerapi_1234567890:300000" }

The following item types are supported: 
* Number, 
* Switch, 
* Dimmer, 
* Contact, 
* String 

PowerDog supports in the PowerAPI the following types for In-Bindings,  which should be  mapped to the following items:

| PowerDog unit | OpenHAB item type | Remark
| ------------- | ----------------- | ------
| V, A, °C, W, l, m/s, km/h | Number, String | 
| % | Number, Switch, Dimmer, Contact, String | In case of Switch, 100% will be mapped to ON; In case of Contact, 100% is mapped to OPEN
| (String) | String | Other values than 'Current_Value' might be only mapped to String

Item configuration examples are therefore:

    Number Power_PV "PV Power [%.1f W]" <sun> (PV) { powerdoglocalapi="<powerdog:impulsecounter_1234567890:300000" }
    Number Power_Supplied "Supplied Power [%.1f W]" <energy> (PV) { powerdoglocalapi="<powerdog:impulsecounter_1372456576:10000" }
    String Power_Supplied_Unit1M { powerdoglocalapi="<powerdog:impulsecounter_1372456576:300000:Unit_1000000" }
    Number Temperature_Outside "Outside temperature [%.1f °C]" <temperature> (Weather) { powerdoglocalapi="<powerdog:onewire_1234567890:300000" }
    Number Temperature_Outside "Outside temperature [%.1f °C]" <temperature> (Weather) { powerdoglocalapi="<powerdog:onewire_1234567890:300000" }
    Number OpenHAB_Alive_2PD "Connection OpenHAB->PowerDog alive [%.1f %%]" <network> { powerdoglocalapi=">pd:powerapi_1234567890:150000" }


## Known Limitations

To reduce load on PowerDog, you are advised to limit querying of the PowerDog variables to not (much) less than every 5 minutes. Otherwise poor performance of the PowerDog (Web) interface including freezing of the box will occur.
For reading counters, the PowerDog Local API does not allow to read the value of a counter, but only the gradient is returned.


## Roadmap & Wishlist

PowerAPI counters are not yet supported due to missing documentation.