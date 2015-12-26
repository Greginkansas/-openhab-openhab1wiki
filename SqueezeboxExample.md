Below you'll find an example configuration for the squeezebox binding to select different radio stations.

 - [Binding](SqueezeboxExample#openhabcfg)
 - [Items](SqueezeboxExample#squeezeitems)
 - [Rules](SqueezeboxExample#squeezerules)
 - [Sitemap](SqueezeboxExample#squeezesitemap) 

![](https://dl.dropboxusercontent.com/u/1781347/wiki/2014-12-18%2017_13_05-openHAB.png)

# openhab.cfg

Here you'll need to configure your players; please make sure that the id's match the name of your logitech configuration. Do not use special characters for the player id.

    ### Squeezebox
    squeeze:server.host     = 192.168.10.59

    squeeze:Bad.id          = 00:04:20:28:65:c7
    squeeze:Bastelzimmer.id = 00:04:20:29:62:0e
    squeeze:Buero.id        = 00:04:20:29:a7:27
    squeeze:Kueche.id       = 00:04:20:28:65:91
    squeeze:Schlafzimmer.id = 00:04:20:2a:37:4b
    squeeze:TV.id           = 00:04:20:23:52:3c
    squeeze:Wohnbereich.id  = 00:04:20:2a:3b:21

# squeeze.items

    /* SqueezeBox */
    Number squeezeSelectedPlayer
    Number squeezeSelectedStation
    Switch squeezePlay

    Switch squeezeBadPower           "Bad" <squeeze> (gPlayerPower, gPlayerPowerOG) { squeeze="Bad:power" }
    Switch squeezeBadPlay            "Bad"                                          { squeeze="Bad:play" }
    Dimmer squeezeBadVolume          "Bad [%.1f %%]" <volume> (gPlayerVolume)       { squeeze="Bad:volume" }

    Switch squeezeBastelzimmerPower  "Gästezimmer" <squeeze> (gPlayerPower, gPlayerPowerOG) { squeeze="Bastelzimmer:power" }
    Switch squeezeBastelzimmerPlay   "Gästezimmer"                                          { squeeze="Bastelzimmer:play" }
    Dimmer squeezeBastelzimmerVolume "Gästezimmer [%.1f %%]" <volume> (gPlayerVolume)       { squeeze="Bastelzimmer:volume" }

    Switch squeezeBueroPower         "Büro" <squeeze> (gPlayerPower, gPlayerPowerOG) { squeeze="Buero:power" }
    Switch squeezeBueroPlay          "Büro"                                          { squeeze="Buero:play" }
    Dimmer squeezeBueroVolume        "Büro [%.1f %%]" <volume> (gPlayerVolume)       { squeeze="Buero:volume" }

    Switch squeezeSchlafzimmerPower  "Schlafzimmer" <squeeze> (gPlayerPower, gPlayerPowerOG) { squeeze="Schlafzimmer:power" }
    Switch squeezeSchlafzimmerPlay   "Schlafzimmer"                                          { squeeze="Schlafzimmer:play" }
    Dimmer squeezeSchlafzimmerVolume "Schlafzimmer [%.1f %%]" <volume> (gPlayerVolume)       { squeeze="Schlafzimmer:volume" }

    Switch squeezeTVPower            "TV" <squeeze> (gPlayerPower, gPlayerPowerEG, gTV) { squeeze="TV:power" }
    Switch squeezeTVPlay             "TV"                                               { squeeze="TV:play" }
    Dimmer squeezeTVVolume           "TV [%.1f %%]" <volume> (gPlayerVolume)            { squeeze="TV:volume" }

    Switch squeezeKuechePower        "Küche" <squeeze> (gPlayerPower, gPlayerPowerEG) { squeeze="Kueche:power" }
    Switch squeezeKuechePlay         "Küche"                                          { squeeze="Kueche:play" }
    Dimmer squeezeKuecheVolume       "Küche [%.1f %%]" <volume> (gPlayerVolume)       { squeeze="Kueche:volume" }

    Switch squeezeWohnbereichPower   "Wohnbereich" <squeeze> (gPlayerPower, gPlayerPowerEG) { squeeze="Wohnbereich:power" }
    Switch squeezeWohnbereichPlay    "Wohnbereich"                                          { squeeze="Wohnbereich:play" }
    Dimmer squeezeWohnbereichVolume  "Wohnbereich [%.1f %%]" <volume> (gPlayerVolume)       { squeeze="Wohnbereich:volume" }

#squeeze.rules
```Xtend
	import org.openhab.core.library.types.*
	import org.openhab.model.script.actions.*
	import org.openhab.action.squeezebox.*

	// Handle squeezebox radio station UI
	rule "SqueezePlayerRadioStation"
	  when 
		Item squeezePlay changed
	  or
		Item squeezeSelectedStation changed
	  or
		Item squeezeSelectedPlayer changed
	  then
		logInfo("squeeze.rules", "SqueezePlayerPlay")
	   
		var String [] players = newArrayList("Bad", "Bastelzimmer", "Buero", "Schlafzimmer", "TV", "Kueche", "Wohnbereich");
		var String[] urls = newArrayList(
		  "http://stream.srg-ssr.ch/drs1/mp3_128.m3u", // Radio SRF1
		  "http://stream.srg-ssr.ch/drs2/mp3_128.m3u", // Radio SRF2
		  "http://stream.srg-ssr.ch/drs3/mp3_128.m3u", // Radio SRF3
		  "http://www.swissgroove.ch/listen.m3u",      // Swiss Groove
		  "http://icecast.argovia.ch/argovia128.m3u",  // Radio Argovia
		  "http://stream.srg-ssr.ch/rsj/mp3_128.m3u",  // Swiss Jazz
		  "http://mp3-live.swr3.de/swr3_m.m3u"         // SWR 3
		  )

		logInfo("squeeze.rules", squeezeSelectedStation.toString)
		
		var stationIndex = ((squeezeSelectedStation.state as DecimalType).intValue - 1)
		var station = urls.get(stationIndex) as String;
		
		var playerIndex = ((squeezeSelectedPlayer.state as DecimalType).intValue - 1) 
		var player = players.get(playerIndex) as String
		
		logInfo("squeeze.rules", player)
		logInfo("squeeze.rules", station)
		 
		if (squeezePlay.state == ON) {
		  squeezeboxPlayUrl(player, station)
		} else {
		  squeezeboxStop(player)
		}

	  end
```
#squeeze.sitemap
	sitemap
	{
		Frame label="System"  {
			Text label="Audio" icon="squeeze"  {
				Frame label="Alle"  {
					Switch item=gPlayerPowerAll label="EG & OG" 				
					Switch item=gPlayerPowerEG label="EG" 				
					Switch item=gPlayerPowerOG label="OG" 		
					Group item=gPlayerPower label="Ein / Aus"  {
						Switch item=squeezeBadPower           				
						Switch item=squeezeBastelzimmerPower  
						Switch item=squeezeBueroPower 	      
						Switch item=squeezeSchlafzimmerPower  
						Switch item=squeezeTVPower 			  
						Switch item=squeezeKuechePower 		  
						Switch item=squeezeWohnbereichPower   
					}
					Group item=gPlayerVolume label="Lautstärke"  {
						Slider item=squeezeBadVolume switchSupport 		    
						Slider item=squeezeBastelzimmerVolume switchSupport 
						Slider item=squeezeBueroVolume switchSupport 		
						Slider item=squeezeSchlafzimmerVolume switchSupport 
						Slider item=squeezeTVVolume switchSupport 		    
						Slider item=squeezeKuecheVolume switchSupport 		
						Slider item=squeezeWohnbereichVolume switchSupport 	
					}
					Slider item=gPlayerVolume label="Lautstärke" 				
				}			
				Frame label="Einzeln"  {				
					Selection item=squeezeSelectedPlayer label="Gerät" mappings=[1="Bad", 2="Gästezimmer", 3="Büro", 4="Schlafzimmer", 5="TV", 6="Küche", 7="Wohnbereich"]
					Selection item=squeezeSelectedStation label="Sender" mappings=[1="SRF 1", 2="SRF 2", 3="SRF 3", 4="Swiss Groove", 5="Argovia", 6="Swiss Jazz", 7="SWR 3"]
					Switch item=squeezePlay label="Stop / Play" mappings=[OFF="Stop", ON="Play"]
					
					Switch item=squeezeBadPower           visibility=[squeezeSelectedPlayer==1]				
					Switch item=squeezeBastelzimmerPower  visibility=[squeezeSelectedPlayer==2]
					Switch item=squeezeBueroPower 	      visibility=[squeezeSelectedPlayer==3]
					Switch item=squeezeSchlafzimmerPower  visibility=[squeezeSelectedPlayer==4]
					Switch item=squeezeTVPower 			  visibility=[squeezeSelectedPlayer==5]
					Switch item=squeezeKuechePower 		  visibility=[squeezeSelectedPlayer==6]
					Switch item=squeezeWohnbereichPower   visibility=[squeezeSelectedPlayer==7]
					
					Slider item=squeezeBadVolume switchSupport 		    visibility=[squeezeSelectedPlayer==1]
					Slider item=squeezeBastelzimmerVolume switchSupport visibility=[squeezeSelectedPlayer==2]
					Slider item=squeezeBueroVolume switchSupport 		visibility=[squeezeSelectedPlayer==3]
					Slider item=squeezeSchlafzimmerVolume switchSupport visibility=[squeezeSelectedPlayer==4]
					Slider item=squeezeTVVolume switchSupport 		    visibility=[squeezeSelectedPlayer==5]
					Slider item=squeezeKuecheVolume switchSupport 		visibility=[squeezeSelectedPlayer==6]
					Slider item=squeezeWohnbereichVolume switchSupport 	visibility=[squeezeSelectedPlayer==7] 			
				}		
			}		
		}	
	}