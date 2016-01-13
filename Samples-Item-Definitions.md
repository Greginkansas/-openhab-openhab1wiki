Samples of Item definitions
* [How to configure a switch to be a pushbutton](Samples-Item-Definitions#how-to-configure-a-switch-to-be-a-pushbutton)
* [How to set up voice control for use with HABDroid](Samples-Item-Definitions#how-to-set-up-voice-control-for-use-with-habdroid)
* [How to get special characters like "%" in a label text](Samples-Item-Definitions#how-to-get-special-characters-like--in-a-label-text)

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
