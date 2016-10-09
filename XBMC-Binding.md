Documentation of the XBMC binding

## Introduction

This page describes the XBMC binding, which allows openhab items to receive realtime updates about information like player state and running media from one or more instances of xbmc.

[![openhab binding for xbmc](http://img.youtube.com/vi/N7_5phTVbo0/0.jpg)](http://www.youtube.com/watch?v=N7_5phTVbo0)

### Installation 

For installation of the binding, please see Wiki page Bindings.

### System requirements

This binding has been developed and tested from XBMC Frodo (v12) to Kodi Jarvis (v16.1).
The binding uses XBMCs JSON-RPC API v6, both the HTTP and the websocket transports. These therefore need to be enabled and need to be reachable from your openHAB host (think firewall, routing etc.)

### Support

If you have any questions about this binding, encounter any problems or think you found a bug, please issue a bug report.

## Global binding configuration

The XBMC binding allows you to define named instances of XBMC in your openhab.cfg. When defining your item binding configuration you can use the name to refer to your instances. In doing so, you can easily change the address by which you XBMC instance can be reached without having to reconfigure all of your bindings. 

The syntax of the binding configuration is like this:
```
xbmc:{instanceName}.{parameter}={value}
```
Where `instanceName` is the name by which you can refer to this instance in your item binding configuration.
`parameter` can be one of the following
* host
* rsPort
* wsPort
* username
* password 
* refreshInterval (available ONLY from 1.9)

### Example OpenHAB v1

    ######################## XBMC Binding ###########################
    
    # Hostname / IP address of your XBMC host
    xbmc:livingRoom.host=192.168.1.6
    
    # Port number for the json rpc service (optional, defaults to 8080)
    xbmc:livingRoom.rsPort=8080
    
    # Port number for the web socket service (optional, defaults to 9090)
    xbmc:livingRoom.wsPort=9090
    
    # Username to connect to XBMC. (optional, defaults to xbmc)
    xbmc:livingRoom.username=xbmc
    
    # Password to connect to XBMC. (optional, defaults to xbmc)
    xbmc:livingRoom.password=xbmc

    (ONLY FROM V1.9 onwards)
    # Refresh interval. Global setting. (optional, defaults to 60000ms = 1 minute)
    xbmc:refreshInterval=60000

### Example OpenHAB v2: xbmc.cfg

    ######################## XBMC Configuration ###########################
    
    # Hostname / IP address of your XBMC host
    livingRoom.host=192.168.1.6
    
    # Port number for the json rpc service (optional, defaults to 8080)
    livingRoom.rsPort=8080
    
    # Port number for the web socket service (optional, defaults to 9090)
    livingRoom.wsPort=9090
    
    # Username to connect to XBMC. (optional, defaults to xbmc)
    livingRoom.username=xbmc
    
    # Password to connect to XBMC. (optional, defaults to xbmc)
    livingRoom.password=xbmc

    (ONLY FROM V1.9 onwards)
    # Refresh interval. Global setting. (optional, defaults to 60000ms = 1 minute)
    refreshInterval=60000


Although you *can* configure one or more instances, it is not strictly necessary to do so. As long as you don't intend to use ports or login data different from the default, you could just as well use this by directly referring to hostnames or IP addresses in you item binding configuration.

## Item binding configuration

### Configuration format

    xbmc="{direction}[{host}|{property}]"

Where
* {direction}: < for incoming (player state, media title etc.), > for outgoing (Send "Play/Pause" or "Stop" etc.) or = for bi-directional (send/receive volume) 
* {host}: When prefixed by a '#' a named instance, otherwise hostname or ip address of your xbmc host.
* {property}: The property to bind to (more on this later)

Some examples:

    xbmc="<[#livingRoom|Player.State]"
    xbmc="<[#livingRoom|Player.Title]"

    xbmc="<[#bedroom|Player.State]"

    xbmc="<[192.168.1.6|Player.State]"
    xbmc="<[xbmc.local|Player.Title]"

    xbmc="=[#livingRoom|Application.Volume]" 

### Available properties (until V1.8)

Currently, the list of properties you can use in your binding configuration is very limited:

Property          | Direction | Description
----------------- |:---------:| ------------
Player.State      | <         | Current player state: 'Play', 'Pause', 'End' (Stopped due to end of content), 'Stop' (Stopped by user). An 'End' state is immediately followed by a 'Stop' state.
Player.Type       | <         | Currently playing type: 'movie', 'episode', 'channel', or 'unknown' for a PVR recording
Player.Title      | <         | Currently playing title: movie name, TV episode name, music track name
Player.ShowTitle  | <         | Currently playing show title: TV show name, empty for other types
Player.Artist     | <         | Currently playing artist (music only)
Player.Album      | <         | Currently playing album (music only)
Player.Label      | <         | Currently playing title (in case of radio streams) 
Player.<???>      | <         | Any other player property supported by the XBMC JSON RPC API
Player.PlayPause  | >         | Play/pause playback
Player.Stop       | >         | Stop playback
Player.Open       | >         | Open a URL for playback
System.Shutdown   | >         | Send OFF to power down the system
System.Suspend    | >         | Send OFF to put the system in Suspend mode
System.Hibernate  | >         | Send OFF to put the system in Hibernate mode
System.Reboot     | >         | Send OFF to reboot the system
System.State      | <         | State of the XBMC application (ON if connected, OFF is XBMC unavailable)
GUI.ShowNotification | >         | Show a notification in the XBMC UI
Application.Volume   | =         | Show and adjust the volume in the XBMC application
PVR.OpenTV       | >         | Open a PVR TV Channel by name for playback
PVR.OpenRadio       | >         | Open a PVR Radio Channel for playback
Screensaver.State    | <         | Current screensaver state: (ON if screensaver active)

### Starting from V1.9 the configuration format is enhanced maintaining full backwards compatibility:

    xbmc="{direction}[{host}|{property}.{command}]"

The list of available {property} is:
* Player (new in V1.9. See below)
* Input (new in V1.9. See below)
* Property (new in V1.9. See below)
* Label (new in V1.9. See below)
* Refresh (new in V1.9. See below)
* System (same as V1.8. See above)
* GUI (same as V1.8. See above)
* Application (same as V1.8. See above)
* PVR (same as V1.8. See above)

#### Property = Player

Command           | Direction | Description
----------------- |:---------:| ------------
State             | <         | Current player state: 'Play', 'Pause', 'End' (Stopped due to end of content), 'Stop' 
Type       | <         | Currently playing type: 'movie', 'episode', 'channel', or 'unknown' for a PVR recording
Title      | <         | Currently playing title: movie name, TV episode name, music track name
ShowTitle  | <         | Currently playing show title: TV show name, empty for other types
Artist     | <         | Currently playing artist (music only)
Album      | <         | Currently playing album (music only)
Label      | <         | Currently playing title (in case of radio streams) 

Command           | Direction | Description
----------------- |:---------:| ------------
PlayPause  | >         | Play/pause playback
Stop       | >         | Stop playback
Open       | >         | Open a URL for playback
Zoom       | >         | Zoom screen
Rotate     | >         | Rotate photo

```
Examples: 
* xbmc=">[livingRoom|Player.PlayStop]"
* xbmc="<[livingRoom|Player.Title]"
```

#### Property = Input

Command           | Direction | Description
------------------- |:---------:| ------------
ExecuteAction.up    | >         | Moves cursor UP 
ExecuteAction.down  | >         | Moves cursor RIGHT
ExecuteAction.left  | >         | Moves cursor LEFT
ExecuteAction.right | >         | Moves cursor RIGHT
ExecuteAction.select | >         | Moves cursor RIGHT
ExecuteAction.back | >         | Moves cursor BACK
ExecuteAction.info | >         | Shows INFO screen 
ExecuteAction.codecinfo | >         | Shows codec information
ExecuteAction.osd | >         | Shows OSD
ExecuteAction.aspectratio | >         | Changes Aspect ratio
ExecuteAction.togglefullscreen | >         | Toggles full screen / Windowed screen
ExecuteAction.contextmenu | >         | Shows context menu
ExecuteAction.???? | >         | Any other command from the following list: http://kodi.wiki/view/JSON-RPC_API/v6#Input.Action (at present 172 commands available)

```
Examples: 
* xbmc=">[livingRoom|Input.ExecuteAction.up]"
* xbmc=">[livingRoom|Input.ExecuteAction.down]"
* xbmc=">[livingRoom|Input.ExecuteAction.togglewatched]"
```

#### Property = Property

Command             | Direction | Description
------------------- |:---------:| ------------
Type                | <         | Gets the media type
Percentage          | <         | Gets the elapsed percentage of the playing media 
Time                | <         | Gets the elapsed time of the playing media
TotalTime           | <         | Gets the total time of the playing media 
CurrentSubtitle     | <         | Gets the current subtitle language of the playing media
????       | <         | Any property from this list: http://kodi.wiki/view/JSON-RPC_API/v6#Player.Property.Name

```
Examples: 
* xbmc="<[livingRoom|Property.Percentage]"
* xbmc="<[livingRoom|Property.TotalTime]"
```

#### Property = Label

Command           | Direction | Description
------------------- |:---------:| ------------
Player.Time    | >         | Moves cursor UP 
System.Uptime  | >         | Moves cursor RIGHT
VideoPlayer.VideoCodec  | >         | Moves cursor LEFT
????.???? | >         | Any other command from the following list: http://kodi.wiki/view/InfoLabels

```
Examples: 
* xbmc=">[livingRoom|Label.Player.Time]"
* xbmc=">[livingRoom|Label.System.Uptime]"
* xbmc=">[livingRoom|Label.VideoPlayer.VideoCodec]"
```

#### Property = Refresh
```
Example: 
* xbmc=">[livingRoom|Refresh]" => Force refresh of incoming properties
```

Incoming properties that are not part of any XBMC notification, will be refreshed with each refresh interval of the binding (default 60 seconds). 

## Example use case

As can be seen in the introduction video, i have a dimmable lamp next to my tv. I use this to light up the room after sunset when i press pause and go back to the previous level on resume, unless the lamp state has been altered while on pause.
This is of course an example very specific to my preferences, but should give you an idea on what can be done.

    import org.openhab.core.library.types.*
    import org.openhab.core.persistence.*
    import org.openhab.model.script.actions.*
    import java.util.Date

    var Number brightnessBeforePause

    rule "Lights on when paused"
    when
            Item LivingRoom_XBMC changed from Play to Pause
    then
            var Date sunsetTime = (G_SUNSET_V.state as DateTimeType).calendar.time
            var Date sunriseTime = (G_SUNRISE_V.state as DateTimeType).calendar.time
            if( now.toDate().after(sunsetTime) || now.toDate().before(sunriseTime)){
                    brightnessBeforePause = LivingRoom_Lamp.state as DecimalType
                    if ( brightnessBeforePause < 50){
                            sendCommand( LivingRoom_Lamp, "50")     
                    }
            }
    end

    rule "Lights off when pause end"
    when
            Item LivingRoom_XBMC changed from Pause to Play
    then
            if (LivingRoom_Lamp.state == 50){               
                    sendCommand( LivingRoom_Lamp, brightnessBeforePause)
            }
    end

## XBMC Binding combined with Wake-On-LAN and Network Health Bindings

The following configuration can be used to power on your XBMC machine using the Wake-On-LAN Binding and power it off using the XBMC Binding. The Network Health binding updates the state after some time if XBMC was powered on or off without using openHAB. You have to insert your own IP and MAC address of course.

    Switch XBMC_Power "XBMC Power" {xbmc=">[#openelec|System.Shutdown]", wol="192.168.1.0#xx:xx:xx:xx:xx:xx", nh="openelec:80"}

## Example of using the Player.Open option

To use the Player.Open command, you would need to define a String item with this binding, then send an Update to this String item. This will cause Openhab to send the Player.Open command to the specified XBMC instance:

### Items

    String XBMC_OpenMedia    "XBMC Player file [%s]"  { xbmc=">[#woonkamer|Player.Open]" }
    Switch XBMC_Radio        "XBMC Start Radio"       

### Rules

    rule "Start Radio"
    when
        Item XBMC_Radio changed to ON
    then
        postUpdate(XBMC_Radio, OFF)
        sendCommand(XBMC_OpenMedia, "http://icecast.omroep.nl/3fm-bb-mp3")
    end

This will introduce a Switch item with an associated rule that will send the Player.Open command when the switch is pressed. 

## Example of using the Application.Volume option

### Item

    Dimmer XBMC_Volume "XBMC Volume [%.1f]" (XBMC) { xbmc="=[#woonkamer|Application.Volume]" }

### Sitemap

    Slider item=XBMC_Volume step=10

### Example of how to set the volume  in the rules script

`sendCommand(XBMC_Volume, "50" )`

## Example of Screensaver.State

### Item

    Switch LG_Screensaver "XBMC Screensaver Living Room" {xbmc="<[#woonkamer|Screensaver.State]"}

### Rule

    rule "turn tv on or off"
	when
		Item LG_Screensaver  changed
	then
		if(LG_Screensaver.state == ON)
		{
			logInfo("screensaver", "turn tv OFF")
			sendCommand(TV_Power,OFF)
                } else{
			logInfo("screensaver", "turn tv ON")
			sendCommand(TV_Power,ON)
		}
    end

## Example of PVR.OpenTV

### Item

    String LG_OpenTVChannel    "Open TV Channel [%s]"  { xbmc=">[#woonkamer|PVR.OpenTV]" }

### rule

    rule "Voice Command open tv channel"
     when
        Item VoiceCommand received command
     then
        sendCommand(LG_OpenTVChannel,VoiceCommand.state.toString)
    end