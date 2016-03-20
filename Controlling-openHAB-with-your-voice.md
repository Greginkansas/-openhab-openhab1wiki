# Controlling openHAB with your voice
This page should give an example how you can control openHAB via natural language without having to write code for every single item you want to control with your voice. 

## Prerequisites

### Voice Recognition systems
You need a working voice recognition system, that sends the recognized text to the VoiceCommand item in openHAB.
The easiest way of doing is by using HABDroid, which works out of the box.

Other possible solutions are Tasker together with AutoVoice of Jasper (on Linux systems).

### Item structure
Your items should follow this naming convention:

    <purpose>_<floorlevel>_<room>_<detail>
    <purpose>    = Shutter, Socket, Light, Temperature,...
    <floorlevel> = GF, FF, SF,...
    <room>       = Bed, Living, Kitchen,...
    <detail>     = Door, Window, ...`

Additionaly there should be one "root"-group which contains all items that should be controllable by voice commands. In the given example this group is named "All"

And you need one String item named VoiceCommand

	String VoiceCommand

## Natural language processing rule (german)

This rule processes German voice commands and should be easily translatable to other languages. 
Basically the rule tries to read the state, the room (and the floorlevel of the room) and the purpose of the item to be changed. Based on this findings the item group "All" is searched for an item named "purpose_floorlevel_room_detail" which accepts the new state.

An additional feature in this rule is the possibility to give responses by TTS (currently only used when the temperature is changed in a room).

	rule "Voice control"
	when
		Item VoiceCommand received command
	then
				var String command = VoiceCommand.state.toString.toLowerCase
		logInfo("Voice.Rec","VoiceCommand received "+command)
		var State newState = null
		// find new state, toggle otherwise (if possible)
		if (command.contains("grad") || command.contains("prozent") || command.contains("dimme")) {
			// extract new state (find the digits in the string)
			var Pattern p = Pattern::compile(".* ([0-9]+) (grad|prozent).*")
			var Matcher m = p.matcher(command)
			if (m.matches()) {
				newState = new StringType(m.group(1).trim())
			}
		}
		else if (command.contains("aus")|| command.contains("ausschalten") || command.contains("beenden")) {
			newState = OFF
		} else if (command.contains("an") || command.contains("ein")|| command.contains("einschalten") || command.contains("starten")) {
			newState = ON
		} else if (command.contains("runterfahren") || command.contains("runter") || command.contains("ab") || command.contains("schließen")) {
			newState = DOWN
		} else if (command.contains("hochfahren") || command.contains("hoch") || command.contains("auf") || command.contains("öffnen")) {
			newState = UP
		} else if (command.contains("rot")) {
			newState = new HSBType(new Color(255, 0, 0));
		} else if (command.contains("blau")) {
			newState = new HSBType(new Color(0, 0, 255));
		} else if (command.contains("weiß")) {
			newState = new HSBType(new Color(255, 255, 255));
		}
		
		// find room
		var String room = null
		var String roomItemPart = null
		var String roomArticle="im"
		if (command.contains("wohnzimmer")) {
			room = "Wohnzimmer"
			roomItemPart = "FF_Living"
		} else if (command.contains("schlafzimmer")) {
			room = "Schlafzimmer"
			roomItemPart = "SF_Bed"
		} else if (command.contains("badezimmer") || command.contains("bad")) {
			room = "Badezimmer"
			roomItemPart = "FF_Toilet"
		} else if (command.contains("arbeitszimmer") || command.contains("büro")) {
			room = "Büro"
			roomItemPart = "FF_Office"
		} else if (command.contains("küche")) {
			room = "Küche"
			roomItemPart = "FF_Kitchen"
			roomArticle = "in der"
		}
		// purpose
		var String itemType=null
		var String itemSubType=""
		var String reply=""
		if (command.contains("licht")) {
			itemType = "Light"
			if (newState instanceof HSBType) {
				if (room=="Schlafzimmer") {
					itemSubType="_Roof"
				} else if (room=="Wohnzimmer") {
					itemSubType="_Closet"
				}
			}
		} else if (command.contains("rolladen") || command.contains("jalousien")) {
			itemType = "Shutter"
			if (newState instanceof StringType) {
				itemType = "Shutter_Pos"
			}
			if (room=="Wohnzimmer") {
				// Unterscheiden zwischen Tür/Fenster
				if (command.contains("tür")) {
					itemSubType = "_Door"
				} else if (command.contains("fenster")) {
					itemSubType = "_Window"
				}
			}
		} else if (command.contains("temperatur")) {
			itemType = "Temperature"
			itemSubType = "_Target"
			reply = "Ok, Temperatur "+roomArticle+" "+room+" auf "+newState+" Grad gesetzt"
		}
		
		if (itemType!=null && (roomItemPart!=null || command.contains("alle")) && newState!=null) {
			logInfo("Voice.Rec", "sending "+newState+" to "+itemType+"_"+roomItemPart+itemSubType)
			if (command.contains("alle")) {
				if (roomItemPart==null)
					roomItemPart=""
				val String itemName = itemType+"_"+roomItemPart+itemSubType
				val State finalState = newState
				logInfo("Voice.Rec","searching for  *"+itemName+"* items")
				All?.allMembers.filter(s | s.name.contains(itemName) && s.acceptedDataTypes.contains(finalState.class)).forEach[item|
					logInfo("Voice.Rec","item  "+item+" found")
					sendCommand(item,finalState.toString)
				]	
			} else {
				sendCommand(itemType+"_"+roomItemPart+itemSubType,newState.toString)
			}
			if (reply!="")
				say(reply)
		}
	end
## Natural language processing rule (English and adapted version)

The code above was a perfect starting point that saved me a lot of time, but it didn't really work for me. The version below is translated to English and slightly adjusted.

    import org.openhab.core.library.types.*
    import org.openhab.core.persistence.* 
    import org.openhab.model.script.actions.*
    import org.openhab.core.types.Command 
    import org.openhab.core.types.*
    import org.openhab.core.items.GenericItem 

    rule "speak"
    when
        Item TTS_Message received command
    then
        executeCommandLine("/home/pi/tts.sh " + TTS_Message.state.toString )
    end

    rule "push debug switch"
    when
        Item DebugSwitch changed from OFF to ON
    then
        sendCommand(VoiceCommand,"Switch off cabinet light in the living")
    end

    rule "VoiceControl" 

    when 
        Item VoiceCommand received command 
    then 
        var String command = VoiceCommand.state.toString.toLowerCase
    //    logInfo("Voice.Rec","VoiceCommand received "+command)
        var State newState = null
        // Status herausfinden, ansonsten togglen
        if (command.contains("degrees") || command.contains("percent") || command.contains("dim")) {
            // Wert setzten=> neuer Status als Zahl extrahieren
    //        var Pattern p = Pattern::compile(".* ([0-9]+) (degrees|percent).*")
    //        var Matcher m = p.matcher(command)
    //        if (m.matches()) {
    //            logInfo("Voice.Rec","found number is "+m.group(1))
    //            newState = new StringType(m.group(1).trim())
    //        } else {
    //            logInfo("Voice.Rec","command does not match")
    //        }
        } else if (command.contains("off") || command.contains("stop")) {
            newState = OFF
        } else if (command.contains("on") || command.contains("start")) {
            newState = ON
        } else if (command.contains("down") || command.contains("close")) {
            newState = DOWN
        } else if (command.contains("up") || command.contains("open")) {
            newState = UP
        }

        // Raum herausfinden
        var String room = null
        var String roomItemPart = null
        var String roomArticle="in the"
        if (command.contains("living")) {
            room = "Living"
            roomItemPart = "GF_Living"
        } else if (command.contains("bedroom")) {
            room = "Bedroom"
            roomItemPart = "FF_Bedroom"
        } else if (command.contains("bathroom") || command.contains("bath")) {
            room = "Bathroom"
            roomItemPart = "FF_Bathroom"
        } else if (command.contains("attic")) {
            room = "Attic"
            roomItemPart = "SF_Attic"
            roomArticle = "at the"
        } else if (command.contains("kitchen")) {
            room = "Kitchen"
            roomItemPart = "FF_Kitchen"
        } 

        // Gewerk
        var String itemType=null
        var String itemSubType=""
        var String reply=""
        if (command.contains("light")) {
            itemType = "Light"
            if (room=="Living") {
                // Unterscheiden zwischen Tür/Fenster
                if (command.contains("little table")) {
                    itemSubType = "_LittleTable"
                } else if (command.contains("toy closet")) {
                    itemSubType = "_ToyCloset"
                } else if (command.contains("cabinet")) {
                    itemSubType = "_Cabinet"
                } else if (command.contains("kitchen top")) {
                    itemSubType = "_KitchenTop"
                }
            }
        } else if (command.contains("shutter")) {
            itemType = "Shutter"
            if (newState instanceof StringType) {
                itemType = "Shutter_Pos"
            }
            if (room=="Living") {
                // Unterscheiden zwischen Tür/Fenster
                if (command.contains("door")) {
                    itemSubType = "_Door"
                } else if (command.contains("window")) {
                    itemSubType = "_Window"
                }
            }
        } else if (command.contains("temperature")) {
            itemType = "Temperature"
            itemSubType = "_Target"
            reply = "Ok, Temperature "+roomArticle+" "+room+" set to "+newState
        } else if (command.contains("radio")) {
            itemType = "Radio"	
        }

    //    logInfo("sending",newState)

        if (itemType != null && (roomItemPart != null || command.contains("all")) && newState != null) {
    //        logInfo("Voice.Rec", "sending "+newState+" to "+itemType+"_"+roomItemPart+itemSubType)
            if (command.contains("all")) {
                if (roomItemPart==null)
                    roomItemPart=""
                val String itemName = itemType+"_"+roomItemPart+itemSubType
                val State finalState = newState
    //            logInfo("Voice.Rec","searching for  *"+itemName+"* items")
                All?.allMembers.filter(s | s.name.contains(itemName) && s.acceptedDataTypes.contains(finalState.class)).forEach[item|
    //                logInfo("Voice.Rec","item  "+item+" found")
                    sendCommand(item.name,finalState.toString)
                ]    
            } else {
                sendCommand(itemType+"_"+roomItemPart+itemSubType,newState.toString)
            }
            if (reply != "")
                sendCommand(TTS_Message,reply)
        }
    end



## TODO
Please feel free to translate this rule into English. If you have a working and well tested solution, please add/replace the rule above.