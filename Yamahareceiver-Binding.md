# Yamahareceiver Binding

Documentation of the Yamahareceiver binding Bundle

## Details
This binding connects openhab with various Yamaha Receivers.

Tested Receivers:
* V473
* V475
* V477
* V479
* V481
* V575
* V671
* V675
* V679
* V773
* V775
* R-N500
* RX-A1030
* RX-A1050 (Aventage)
* RX-A2020
* RX-A3000
* RX-A810
* RX-S601D
* RX-V581
* RX-V673
* RX-V677
* RX-V771
* HTR-4065
* HTP-4069

Please add successfully tested receivers!

## Configuration

inside openhab.cfg you need only the host definition:

     yamahareceiver:<uid>.host=192.168.1.1 

<uid> represent your instance name inside your items list
right from the equal sign you must define the ip address from your receiver.

items:

     {yamahareceiver="uid=living, zone=main, bindingType=power"}

allowed zone entries:

* `main`: Main Zone
* `zone2`: Zone 2
* `zone3`: Zone 3
* `zone4`: Zone 4

depends on the real zones implemented on your receiver.

allowed bindingTypes:

* `power`: Openhab Type `Switch`, Switches The Receiver ON or OFF (ON only works if the Receiver's settings are configured to react on this signal while in the OFF state!  This setting is sometimes labeled "Network Standby" and must be enabled.)

* `mute`: Openhab Type `Switch`, Mute or Unmute the receiver
* `volume`: Openhab Type `Dimmer`, Set's the receivers Volume percent Value.
* `input`: Openhab Type `String`, Set's the input selection, depends on your receiver's real inputs
examples: HDMI1, HDMI2, AV4, TUNER, NET RADIO, etc.
* `surroundProgram`: Openhab Type `String`, Set's the surround Mode
examples: 2ch Stereo, 7ch Stereo, Hall in Munic, Straight, Surround Decoder
 
## Example

     openhab.cfg

     #Yamaha Receiver 
     yamahareceiver:living.host=192.168.1.1
 
     .items

     Switch Yamaha_Power         "Power [%s]"         <tv>    { yamahareceiver="uid=living, zone=main,  bindingType=power" }
     Dimmer Yamaha_Volume         "Volume [%.1f %%]"             { yamahareceiver="uid=living, zone=main, bindingType=volumePercent" }
     Switch Yamaha_Mute             "Mute [%s]"                 { yamahareceiver="uid=living, zone=main, bindingType=mute" }
     String Yamaha_Input         "Input [%s]"                 { yamahareceiver="uid=living, zone=main, bindingType=input" } 
     String Yamaha_Surround         "surround [%s]"             { yamahareceiver="uid=living, zone=main, bindingType=surroundProgram" } 
     Number Yamaha_NetRadio  "Net Radio" <netRadio> { yamahareceiver="uid=living, zone=main, bindingType=netRadio" }
 
     .sitemap

     Selection item=Yamaha_NetRadio label="Sender" mappings=[1="N Joy", 2="Radio Sport", 3="RDU", 4="91ZM", 5="Hauraki"]
     Selection item=Yamaha_Input mappings=[HDMI1="BlueRay",HDMI2="Satellite","NET RADIO"="NetRadio",TUNER="Tuner"]
     Selection item=Yamaha_Surround label="Surround Mode" mappings=["2ch Stereo"="2ch","7ch Stereo"="7ch"]

the tricky thing are the `"` around `NET RADIO`, this Key (left from the equal sign) is a value that must be send to the receiver **with** the space inside. If you omit the `"` the binding sends only the `NET` and the receiver do's nothing. 
Same are in surround definition!