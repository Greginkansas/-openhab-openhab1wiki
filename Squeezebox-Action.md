# Squeezebox Action

Interact directly with your Squeezebox devices from within rules and scripts. In order to use these actions you must also install the **org.openhab.io.squeezeserver** bundle and configure the 'squeeze' properties in openhab.cfg.

## Getting started

See the [[Squeezebox Binding|Squeezebox binding]] section for more details about the Squeezebox binding and how to configure it correctly. Be aware that the MAC address for each player is case sensitive.

There are three "Squeezebox" addons that you need to put into your addons folder.

* org.openhab.action.squeezebox-1.8.2
* org.openhab.binding.squeezebox-1.8.2
* org.openhab.io.squeezeserver-1.8.2

The 'id' you specify in your openhab.cfg will be used to identify which player to perform the specified action on.

For example in openhab.cfg `squeeze:Kitchen_Player.id=de:ad:be:ef:12:34` would be identified as `Kitchen_Player` in your rules.

## Send voice notifications to your Squeezebox devices; 

You need to have filled in the `ttsurl` details within the Squeezebox section in the openhab.cfg file. Given the changes Google have made to their TTS usage allowances, you might have better luck registering for a key at http://www.voicerss.org/api/ 

You can check you have the url and api working by pasting it into a browser with some text at the end

```
https://api.voicerss.org/?key=YOUR_KEY_GOES_HERE&f=44khz_16bit_stereo&hl=en-gb&src=This is Major Tom to Ground Control
```
Then you can use the action in your rules however you want

#### Send an announcement to the specified player using the current volume

```java
// squeezeboxSpeak(String playerId, String message)

squeezeboxSpeak("Kitchen_Player", "This is Major Tom to Ground Control")
```
#### Send an announcement to the specified player at the specified volume

```java
// squeezeboxSpeak(String playerId, String message, int volume)

squeezeboxSpeak("Kitchen_Player", "I'm stepping through the door", 100)
```

#### Send an announcement to the specified player at the specified volume, if `resumePlayback=true` resume to actual playlist after finishing message.

You might have to tweak some settings on your Squeezebox server and player regarding what defeats what when you add a song in the middle of a playlist.

```java
// squeezeboxSpeak(String playerId, String message, int volume, bool resumePlayback)

squeezeboxSpeak("Kitchen_Player", "And I'm floating in a most peculiar way", 100, false)
```

### Generating dynamic strings

```java
squeezeboxSpeak("Kitchen_Player"," temperature outside is " + Weather_Temperature.state.format("%d") + " degrees celsius",75,true)
```

## Play a URL on one of your Squeezebox devices (e.g. start a radio stream when you wake up in the morning);
- `squeezeboxPlayUrl(String playerId, String url)`: Plays the URL on the specified player using the current volume
- `squeezeboxPlayUrl(String playerId, String url, int volume)`: Plays the URL on the specified player at the specified volume

## Standard Squeezebox actions for controlling your devices;
- `squeezeboxPower(String playerId, boolean power)`
- `squeezeboxMute(String playerId, boolean mute)`
- `squeezeboxVolume(String playerId, int volume)`
- `squeezeboxPlay(String playerId)`
- `squeezeboxPause(String playerId)`
- `squeezeboxStop(String playerId)`
- `squeezeboxNext(String playerId)`
- `squeezeboxPrev(String playerId)`

See also [[Core Actions|Actions]].