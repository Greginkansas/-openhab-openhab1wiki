Documentation of the Onkyo AV Receiver Binding

## Introduction

Binding should be compatible with Onkyo AV receivers which support ISCP (Integra Serial Control Protocol) over Ethernet (eISCP) and Serial port (since version 1.9).

For installation of the binding, please see Wiki page [[Bindings]].

## Binding Configuration

First of all you need to introduce your Onkyo AV receiver's in the openhab.cfg file (in the folder '${openhab_home}/configurations').

    ################################# Onkyo  Binding ######################################
    
    # Host of the first Onkyo device to control 
    # onkyo:<OnkyoId1>.host=
    # Port of the Onkyo to control (optional, defaults to 60128)
    # onkyo:<OnkyoId1>.port=
    
    # Host of the second Onkyo device to control 
    # onkyo:<OnkyoId2>.host=
    # Port of the Onkyo to control (optional, defaults to 60128)
    # onkyo:<OnkyoId2>.port=

The `onkyo:<OnkyoId1>.host` value is the ip address of the Onkyo AV receiver. 

The `onkyo:<OnkyoId1>.port` value is TCP port address of the the receiver. Port value is optional parameter.

Since Version 1.9, you can also control receivers with Serial port
    # Serial Port of the third Onkyo device to control 
    # onkyo:<OnkyoId3>.serialPortName=
 

Examples, how to configure your receiver device:

    onkyo:hometheater.host=192.168.1.100
    onkyo:hometheater.port=60128

Or

onkyo:hometheater.serialPortName=/dev/ttyUSB0

## Item Binding Configuration

In order to bind an item to the device, you need to provide configuration settings. The easiest way to do so is to add some binding information in your item file (in the folder configurations/items`). The syntax of the binding configuration strings accepted is the following:

    onkyo="<openHAB-command>:<device-id>:<device-command>[,<openHAB-command>:<device-id>:<device-command>][,...]"

where parts in brackets [] signify an optional information.

The **openHAB-command** corresponds OpenHAB command. 

The **device-id** corresponds device which is introduced in openhab.cfg.

The **device-command** corresponds Onkyo AV receiver command. See complite list below.


Examples, how to configure your items:

    Switch onkyoPower  {onkyo="ON:hometheater:POWER_ON, OFF:hometheater:POWER_OFF"}
    Dimmer onkyoVolume {onkyo="INCREASE:hometheater:VOLUME_UP, DECREASE:hometheater:VOLUME_DOWN"}


### List of predefined Onkyo AV receiver commands

- **Main zone**
  - _Power_
    - POWER_OFF
    - POWER_ON
    - POWER_QUERY
  - _Mute_
    - UNMUTE
    - MUTE
    - MUTE_QUERY
  - _Volume_
    - VOLUME_UP
    - VOLUME_DOWN
    - VOLUME_QUERY
    - VOLUME_SET
    - SET_VOLUME
    - VOLUME
  - _Source_
    - SOURCE_DVR
    - SOURCE_VCR
    - SOURCE_SATELLITE
    - SOURCE_CABLE
    - SOURCE_GAME
    - SOURCE_AUXILIARY
    - SOURCE_AUX
    - SOURCE_VIDEO5
    - SOURCE_AUX2
    - SOURCE_COMPUTER
    - SOURCE_PC
    - SOURCE_BLURAY
    - SOURCE_DVD
    - SOURCE_TAPE1
    - SOURCE_TAPE2
    - SOURCE_PHONO
    - SOURCE_CD
    - SOURCE_FM
    - SOURCE_AM
    - SOURCE_TUNER
    - SOURCE_MUSICSERVER
    - SOURCE_INTERETRADIO
    - SOURCE_USB
    - SOURCE_USB_BACK
    - SOURCE_NETWORK
    - SOURCE_MULTICH
    - SOURCE_SIRIUS
    - SOURCE_UP
    - SOURCE_DOWN
    - SOURCE_QUERY
    - SOURCE_SET
    - SET_SOURCE
    - SOURCE
  - _Video Wide_
    - VIDEO_WIDE_AUTO
    - VIDEO_WIDE_43
    - VIDEO_WIDE_FULL
    - VIDEO_WIDE_ZOOM
    - VIDEO_WIDE_WIDEZOOM
    - VIDEO_WIDE_SMARTZOOM
    - VIDEO_WIDE_NEXT
    - VIDEO_WIDE_QUERY
  - _Listen Mode_
    - LISTEN_MODE_STEREO
    - LISTEN_MODE_ALCHANSTEREO
    - LISTEN_MODE_AUDYSSEY_DSX
    - LISTEN_MODE_PLII_MOVIE_DSX
    - LISTEN_MODE_PLII_MUSIC_DSX
    - LISTEN_MODE_PLII_GAME_DSX
    - LISTEN_MODE_NEO_CINEMA_DSX
    - LISTEN_MODE_NEO_MUSIC_DSX
    - LISTEN_MODE_NEURAL_SURROUND_DSX
    - LISTEN_MODE_NEURAL_DIGITAL_DSX
    - LISTEN_MODE_UP
    - LISTEN_MODE_DOWN
    - LISTEN_MODE_QUERY
- **Zone 2**
  - _Power_
    - ZONE2_POWER_OFF
    - ZONE2_POWER_ON
    - ZONE2_POWER_QUERY
  - _Mute_
    - ZONE2_UNMUTE
    - ZONE2_MUTE
    - ZONE2_MUTE_QUERY
  - _Volume_
    - ZONE2_VOLUME_UP
    - ZONE2_VOLUME_DOWN
    - ZONE2_VOLUME_QUERY
    - ZONE2_VOLUME_SET
    - ZONE2_SET_VOLUME
    - ZONE2_VOLUME
  - _Source_
    - ZONE2_SOURCE_DVR
    - ZONE2_SOURCE_VCR
    - ZONE2_SOURCE_SATELLITE
    - ZONE2_SOURCE_CABLE
    - ZONE2_SOURCE_GAME
    - ZONE2_SOURCE_AUXILIARY
    - ZONE2_SOURCE_AUX
    - ZONE2_SOURCE_VIDEO5
    - ZONE2_SOURCE_AUX2
    - ZONE2_SOURCE_COMPUTER
    - ZONE2_SOURCE_PC
    - ZONE2_SOURCE_BLURAY
    - ZONE2_SOURCE_DVD
    - ZONE2_SOURCE_TAPE1
    - ZONE2_SOURCE_TAPE2
    - ZONE2_SOURCE_PHONO
    - ZONE2_SOURCE_CD
    - ZONE2_SOURCE_FM
    - ZONE2_SOURCE_AM
    - ZONE2_SOURCE_TUNER
    - ZONE2_SOURCE_MUSICSERVER
    - ZONE2_SOURCE_INTERETRADIO
    - ZONE2_SOURCE_USB
    - ZONE2_SOURCE_USB_BACK
    - ZONE2_SOURCE_NETWORK
    - ZONE2_SOURCE_MULTICH
    - ZONE2_SOURCE_SIRIUS
    - ZONE2_SOURCE_UP
    - ZONE2_SOURCE_DOWN
    - ZONE2_SOURCE_QUERY
    - ZONE2_SOURCE_SET
    - ZONE2_SET_SOURCE
    - ZONE2_SOURCE
- **Zone 3**
  - _Power_
    - ZONE3_POWER_OFF
    - ZONE3_POWER_ON
    - ZONE3_POWER_QUERY
  - _Mute_
    - ZONE3_UNMUTE
    - ZONE3_MUTE
    - ZONE3_MUTE_QUERY
  - _Volume_
    - ZONE3_VOLUME_UP
    - ZONE3_VOLUME_DOWN
    - ZONE3_VOLUME_QUERY
    - ZONE3_VOLUME_SET
    - ZONE3_SET_VOLUME
    - ZONE3_VOLUME
  - _Source_
    - ZONE3_SOURCE_DVR
    - ZONE3_SOURCE_VCR
    - ZONE3_SOURCE_SATELLITE
    - ZONE3_SOURCE_CABLE
    - ZONE3_SOURCE_GAME
    - ZONE3_SOURCE_AUXILIARY
    - ZONE3_SOURCE_AUX
    - ZONE3_SOURCE_VIDEO5
    - ZONE3_SOURCE_AUX2
    - ZONE3_SOURCE_COMPUTER
    - ZONE3_SOURCE_PC
    - ZONE3_SOURCE_BLURAY
    - ZONE3_SOURCE_DVD
    - ZONE3_SOURCE_TAPE1
    - ZONE3_SOURCE_TAPE2
    - ZONE3_SOURCE_PHONO
    - ZONE3_SOURCE_CD
    - ZONE3_SOURCE_FM
    - ZONE3_SOURCE_AM
    - ZONE3_SOURCE_TUNER
    - ZONE3_SOURCE_MUSICSERVER
    - ZONE3_SOURCE_INTERETRADIO
    - ZONE3_SOURCE_USB
    - ZONE3_SOURCE_USB_BACK
    - ZONE3_SOURCE_NETWORK
    - ZONE3_SOURCE_MULTICH
    - ZONE3_SOURCE_SIRIUS
    - ZONE3_SOURCE_UP
    - ZONE3_SOURCE_DOWN
    - ZONE3_SOURCE_QUERY
    - ZONE3_SOURCE_SET
    - ZONE3_SET_SOURCE
    - ZONE3_SOURCE
- **Net/USB**
  - _Operations_
    - NETUSB_OP_PLAY
    - NETUSB_OP_STOP
    - NETUSB_OP_PAUSE
    - NETUSB_OP_TRACKUP
    - NETUSB_OP_TRACKDWN
    - NETUSB_OP_FF
    - NETUSB_OP_REW
    - NETUSB_OP_REPEAT
    - NETUSB_OP_RANDOM
    - NETUSB_OP_DISPLAY
    - NETUSB_OP_RIGHT
    - NETUSB_OP_LEFT
    - NETUSB_OP_UP
    - NETUSB_OP_DOWN
    - NETUSB_OP_SELECT
    - NETUSB_OP_1
    - NETUSB_OP_2
    - NETUSB_OP_3
    - NETUSB_OP_4
    - NETUSB_OP_5
    - NETUSB_OP_6
    - NETUSB_OP_7
    - NETUSB_OP_8
    - NETUSB_OP_9
    - NETUSB_OP_0
    - NETUSB_OP_DELETE
    - NETUSB_OP_CAPS
    - NETUSB_OP_SETUP
    - NETUSB_OP_RETURN
    - NETUSB_OP_CHANUP
    - NETUSB_OP_CHANDWN
    - NETUSB_OP_MENU
    - NETUSB_OP_TOPMENU
  - _Song Info_
    - NETUSB_SONG_ARTIST_QUERY
    - NETUSB_SONG_ALBUM_QUERY
    - NETUSB_SONG_TITLE_QUERY
    - NETUSB_SONG_ELAPSEDTIME_QUERY
    - NETUSB_SONG_TRACK_QUERY
    - NETUSB_PLAY_STATUS_QUERY

## Advanced commands

If you want to use commands that are not predefined by the binding you can use them with a shape # as a prefix.

example 1:
    Dimmer volume { onkyo="INCREASE:hometheater:VOLUME_UP, DECREASE:hometheater:VOLUME_DOWN, *:hometheater:#MVL%02X" }


openhab send volume INCREASE -> binding send VOLUME_UP (eISCP command=MVLUP)

openhab send volume DECREASE -> binding send VOLUME_DOWN (eISCP command=MVLDOWN)

openhab send volume 30 -> binding send eISCP command MVL1E (set volume level to 30)

example 2:

    Number zone4Selector { onkyo="*:hometheater:#SL4%02X" }

example 3:

    Switch onkyoPower { onkyo="*:hometheater:#PWR%02X" }

A list of all commands that are supported by Onkyo's eISCP can be found here:
http://blog.siewert.net/?cat=18

Be aware that openhab uses decimal numbers but ISCP uses hex.
For example: The documentation says "NET" (as a source) is the value "2B". You need to translate this from HEX to DEC for openhab. (2B = 43)
items:

    Number onkyoZ2Selector "Source [%d]" {onkyo="INIT:avr:#SLZQSTN, *:avr:#SLZ%02X"}
    
sitemap:

    Selection item=onkyoZ2Selector label="Source" mappings=[127=OFF, 43=NET, 1=SAT]

## Limitations

- NETUSB_SONG_ELAPSEDTIME_QUERY - NET/USB Time Info
- Elapsed time/Track Time Max 99:59
- NETUSB_SONG_TRACK_QUERY - NET/USB Track Info
- Current Track/Toral Track Max 9999
- NJA - NET/USB Jacket Art
- Album Cover cannot be processed yet.

## Demo

onkyo.items:

    //
    // Main
    //
    // Power
    Switch onkyoPower          "Power"                   {onkyo="INIT:hometheater:POWER_QUERY, ON:hometheater:POWER_ON, OFF:hometheater:POWER_OFF"}
    // Sleep
    Number onkyoSleep          "Sleep Timer [%d Min]"    {onkyo="INIT:hometheater:#SLPQSTN, 0:hometheater:#SLPOFF, *:hometheater:#SLP%02X, 0:hometheater:#SLPOFF"}
    // Mute
    Switch onkyoMute           "Mute"                    {onkyo="INIT:hometheater:MUTE_QUERY, ON:hometheater:MUTE, OFF:hometheater:UNMUTE"}
    // Volume
    Dimmer onkyoVolume         "Volume [%d]"             {onkyo="INIT:hometheater:VOLUME_QUERY, INCREASE:hometheater:VOLUME_UP, DECREASE:hometheater:VOLUME_DOWN, *:hometheater:VOLUME_SET"}
    //Source
    Number onkyoSource         "Source"                  {onkyo="INIT:hometheater:SOURCE_QUERY, INCREASE:hometheater:SOURCE_UP, DECREASE:hometheater:SOURCE_DOWN, *:hometheater:SOURCE_SET"}
    //Video Modes
    Number onkyoVideoWide      "Video Wide Mode"         {onkyo="INIT:hometheater:VIDEO_WIDE_QUERY, INCREASE:hometheater:VIDEO_WIDE_NEXT, *:hometheater:#VWM%02X"}
    Number onkyoVideoPicture   "Video Picture Mode"      {onkyo="INIT:hometheater:#VPMQSTN, INCREASE:hometheater:#VPMUP, *:hometheater:#VPM%02X"}
    //Audio Mode
    Number onkyoListenMode     "Listen Mode"             {onkyo="INIT:hometheater:LISTEN_MODE_QUERY, INCREASE:hometheater:LISTEN_MODE_UP, DECREASE:hometheater:LISTEN_MODE_DOWN, *:hometheater:#LMD%02X"}
    Switch onkyoAudysseyDynEQ  "Audysses Dynamic EQ"     {onkyo="INIT:hometheater:#ADQQSTN, OFF:hometheater:#ADQ00, ON:hometheater:#ADQ01"}
    Number onkyoAudysseyDynVol "Audysses Dynamic Volume" {onkyo="INIT:hometheater:#ADVQSTN, INCREASE:hometheater:#ADVUP, *:hometheater:#ADV%02X"}
    //Information
    String onkyoAudio          "Audio [%s]"              {onkyo="INIT:hometheater:#IFAQSTN"}
    String onkyoVideo          "Video [%s]"              {onkyo="INIT:hometheater:#IFVQSTN"}
    // Display
    Number onkyoDisplayMode    "Display Mode"            {onkyo="INIT:hometheater:#DIFQSTN, INCREASE:hometheater:#DIFTG, *:hometheater:#DIF%02X"}
    Number onkyoDimmerLevel    "Display Dimmer Level"    {onkyo="INIT:hometheater:#DIMQSTN, INCREASE:hometheater:#DIMDIM, *:hometheater:#DIM%02X"}
    
    //
    // Zone 2
    //
    // Power
    Switch onkyoZ2Power   "Power"       {onkyo="INIT:hometheater:ZONE2_POWER_QUERY, ON:hometheater:ZONE2_POWER_ON, OFF:hometheater:ZONE2_POWER_OFF"}
    // Mute
    Switch onkyoZ2Mute    "Mute"        {onkyo="INIT:hometheater:ZONE2_MUTE_QUERY:, ON:hometheater:ZONE2_MUTE, OFF:hometheater:ZONE2_UNMUTE"}
    // Volume
    Dimmer onkyoZ2Volume  "Volume [%d]" {onkyo="INIT:hometheater:ZONE2_VOLUME_QUERY, INCREASE:hometheater:ZONE2_VOLUME_UP, DECREASE:hometheater:ZONE2_VOLUME_DOWN, *:hometheater:ZONE2_VOLUME_SET"}
    //Source
    Number onkyoZ2Source  "Source"      {onkyo="INIT:hometheater:ZONE2_SOURCE_QUERY, INCREASE:hometheater:ZONE2_SOURCE_UP, DECREASE:hometheater:ZONE2_SOURCE_DOWN, *:hometheater:ZONE2_SOURCE_SET"}
    
    //
    // Zone 3
    //
    // Power
    Switch onkyoZ3Power  "Power"       {onkyo="INIT:hometheater:ZONE3_POWER_QUERY, ON:hometheater:ZONE3_POWER_ON, OFF:hometheater:ZONE3_POWER_OFF"}
    // Mute
    Switch onkyoZ3Mute   "Mute"        {onkyo="INIT:hometheater:ZONE3_MUTE_QUERY:, ON:hometheater:ZONE3_MUTE, OFF:hometheater:ZONE3_UNMUTE"}
    // Volume
    Dimmer onkyoZ3Volume "Volume [%d]" {onkyo="INIT:hometheater:ZONE3_VOLUME_QUERY, INCREASE:hometheater:ZONE3_VOLUME_UP, DECREASE:hometheater:ZONE3_VOLUME_DOWN, *:hometheater:ZONE3_VOLUME_SET"}
    //Source
    Number onkyoZ3Source "Source"      {onkyo="INIT:hometheater:ZONE3_SOURCE_QUERY, INCREASE:hometheater:ZONE3_SOURCE_UP, DECREASE:hometheater:ZONE3_SOURCE_DOWN, *:hometheater:ZONE3_SOURCE_SET"}
    
    //
    // NET/USB
    //
    // Controls
    Switch onkyoNETPlay      "Play"             { onkyo="ON:hometheater:NETUSB_OP_PLAY", autoupdate="false"}
    Switch onkyoNETPause     "Pause"            { onkyo="ON:hometheater:NETUSB_OP_PAUSE", autoupdate="false"}
    Switch onkyoNETStop      "Stop"             { onkyo="ON:hometheater:NETUSB_OP_STOP", autoupdate="false"}
    Switch onkyoNETTrackUp   "Track Up"         { onkyo="ON:hometheater:NETUSB_OP_TRACKUP", autoupdate="false"}
    Switch onkyoNETTrackDown "Track Down"       { onkyo="ON:hometheater:NETUSB_OP_TRACKDWN", autoupdate="false"}
    Switch onkyoNETFF        "Fast Forward"     { onkyo="ON:hometheater:NETUSB_OP_FF", autoupdate="false"}
    Switch onkyoNETREW       "Rewind"           { onkyo="ON:hometheater:NETUSB_OP_REW", autoupdate="false"}
    Number onkyoNETService   "Service"          { onkyo="INIT:hometheater:#NSVQST, *:hometheater:#NSV%02X0"}
    Number onkyoNETList      "Select List Item" { onkyo="*:hometheater:#NLSL%01X"}
    // Information
    String onkyoNETArtist     "Artist [%s]"      {onkyo="INIT:hometheater:NETUSB_SONG_ARTIST_QUERY"}
    String onkyoNETAlbum      "Album [%s]"       {onkyo="INIT:hometheater:NETUSB_SONG_ALBUM_QUERY"}
    String onkyoNETTitle      "Title [%s]"       {onkyo="INIT:hometheater:NETUSB_SONG_TITLE_QUERY"}
    String onkyoNETTrack      "Track [%s]"       {onkyo="INIT:hometheater:NETUSB_SONG_TRACK_QUERY"}
    String onkyoNETTime       "Time [%s]"        {onkyo="INIT:hometheater:NETUSB_SONG_ELAPSEDTIME_QUERY"}
    String onkyoNETPlayStatus "Play Status [%s]" {onkyo="INIT:hometheater:NETUSB_PLAY_STATUS_QUERY"}

onkyo.sitemap:

    sitemap onkyo label="Onkyo Demo"
    {
        Frame label="Zones" {
            Text label="Main" icon="sofa" {
                Frame label="Power" {
                    Switch    item=onkyoPower
                    Selection item=onkyoSleep mappings=[0=Off, 5="5 Min", 10="10 Min", 15="15 Min", 30="30 Min", 77="77 Min", 90="90 Min"]
                }
                Frame label="Volume" {
                    Switch item=onkyoMute
                    Slider item=onkyoVolume
                }
                Frame label="Source" {
                    Selection item=onkyoSource mappings=[0="VCR/DVR", 1="CBL/SAT", 2=GAME, 5=PC, 16="BD/DVD", 35=CD, 43="NET/USB", 45=Airplay, 127=OFF]
                }
                Frame label="Video Modes" {
                    Selection item=onkyoVideoWide label="Video Wide" mappings=[0=Auto, 1="4:3", 2=Full, 3=Zoom, 4="Wide Zoom", 5="Smart Zoom"]
                    Selection item=onkyoVideoPicture label="Video Picture" mappings=[0=Trough, 1=Custom, 2=Cinema, 3=Game, 5="ISF Day", 6="ISF Night", 7="Streaming", 8=Direct]
                }
                Frame label="Audio Modes" {
                    Selection item=onkyoListenMode mappings=[0=Stereo, 1=Direct, 2=Surround, 15=Mono, 31="Whole House Mode", 66="THX Cinema", 31="Whole House"]
                    Switch    item=onkyoAudysseyDynEQ
                    Selection item=onkyoAudysseyDynVol mappings=[0=OFF, 1=Low, 2=Mid, 3=High]
                }
                Frame label="Information" {
                    Text item=onkyoAudio
                    Text item=onkyoVideo
                }
                Frame label="Display" {
                    Selection item=onkyoDisplayMode mappings=[0="Source + Vol", 2="Digital Format (temporary)", 3="Video Format (temporary)"]
                    Selection item=onkyoDimmerLevel mappings=[0="Bright", 1="Dim", 2="Dark", 3="Shut-Off", 8="Bright & LED OFF"]
                }
            }
            Text label="Zone 2" icon="bedroom" {
                Frame label="Power" {
                   Switch item=onkyoZ2Power
                }
                Frame label="Volume" {
                    Switch item=onkyoZ2Mute
                    Slider item=onkyoZ2Volume
                }
                Frame label="Source" {
                    Selection  item=onkyoZ2Source label="Source Selection" mappings=[0="VCR/DVR", 1="CBL/SAT", 2=GAME, 5=PC, 16="BD/DVD", 35=CD, 43="NET/USB", 45=Airplay, 127=OFF]
                }
            }
    
            Text label="Zone 3" icon="bath" {
                Frame label="Power" {
                   Switch item=onkyoZ3Power
                }
                Frame label="Volume" {
                    Switch item=onkyoZ3Mute
                    Slider item=onkyoZ3Volume
                }
                Frame label="Source" {
                    Selection  item=onkyoZ3Source label="Source Selection" mappings=[0="VCR/DVR", 1="CBL/SAT", 2=GAME, 5=PC, 16="BD/DVD", 35=CD, 43="NET/USB", 45=Airplay, 127=OFF]
                }
            }
            Text label="NET/USB" icon="video" {
                Frame label="Controls" {
                    Switch    item=onkyoNETPlay
                    Switch    item=onkyoNETPause
                    Switch    item=onkyoNETStop
                    Switch    item=onkyoNETTrackUp
                    Switch    item=onkyoNETTrackDown
                    Switch    item=onkyoNETFF
                    Switch    item=onkyoNETREW
                    Selection item=onkyoNETService mappings=[0="Media Server (DLNA)", 1=Favorite, 2=vTuner, 3=SIRIUS, 6="Last.fm", 14=TuneIn Radio]
                    Selection item=onkyoNETList    mappings=[0="1", 1="2", 2="3", 3="4", 4="5", 5="6", 6="7", 7="8", 8="9", 9="10"]
                }
                Frame label="Information" {
                    Text item=onkyoNETArtist
                    Text item=onkyoNETAlbum
                    Text item=onkyoNETTitle
                    Text item=onkyoNETTrack
                    Text item=onkyoNETTime
                }
            }
    
    
        }
    }

###Screenshots
Main: 
![alt text](http://wiki.openhab-samples.googlecode.com/hg/screenshots/onkyo_main.png "Onkyo Main")

Zone: 
![alt text](http://wiki.openhab-samples.googlecode.com/hg/screenshots/onkyo_zone2.PNG "Onkyo Zone")

Net: 
![alt text](http://wiki.openhab-samples.googlecode.com/hg/screenshots/onkyo_net.PNG "Onkyo Net")