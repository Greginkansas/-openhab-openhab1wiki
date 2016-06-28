Samples of Item definitions
* [How to configure a switch to be a pushbutton](Samples-Item-Definitions#how-to-configure-a-switch-to-be-a-pushbutton)
* [How to set up voice control for use with HABDroid](Samples-Item-Definitions#how-to-set-up-voice-control-for-use-with-habdroid)
* [How to get special characters like "%" in a label text](Samples-Item-Definitions#how-to-get-special-characters-like--in-a-label-text)
* [How to use HTTP binding to remotely control devices with a web interface](Samples-Item-Definitions#how-to-use-HTTP-binding)


### How to configure a switch to be a pushbutton

[German Thread](http://knx-user-forum.de/openhab/27123-einfacher-taster-openhab.html)

Item:

    Switch Garage_Gate { binding="xxx", autoupdate="false"}

Sitemap:

    Switch item=Garage_Gate label="Garage" mappings=[ON="Go!"]
The magic happens with `autoupdate="false"` which keeps the state even an ON command has been received. This way, it's always off unless you explicitly post an update to this item.


### How to set up voice control for use with HABDroid
The following example shows an item called `test_item` being turned on by issuing the voice command in HABDroid

Item:

    String VoiceCommand

Rule:

    rule "test example rule name"
    when
                    Item VoiceCommand received command test
            then
                    test_item.sendCommand(ON)
    end

### How to get special characters like "%" in a label text

    Number Humidity "Humidity [%.1f %%]"

### How to use HTTP binding to remotely control devices with a web interface

LG Hombot robot cleaner:

``String Hombot   "Luigi, der Hombot"                     <luigi>         (Status)   { http=">[1:GET:http://192.168.178.52:6260/json.cgi?%%7b%%22COMMAND%%22:%%22CLEAN_START%%22%%7d] >[0:GET:http://192.168.178.52:6260/json.cgi?%%7b%%22COMMAND%%22:%%22PAUSE%%22%%7d] >[2:GET:http://192.168.178.52:6260/json.cgi?%%7b%%22COMMAND%%22:%%22HOMING%%22%%7d] <[http://192.168.178.52:6260/status.html:5000:REGEX(.*<b>Robot-state</b>: <status>(.+)</status>.*)]", autoupdate="false" }``

Dreambox satellite receiver:

``Switch Dreambox "Dreambox [%s]"                         <video>         (EG_Wohnen,Status) { http=">[ON:POST:http://dm500hd/web/powerstate?newstate=4] >[OFF:POST:http://dm500hd/web/powerstate?newstate=5]" }``
