
* [Worx Landroid](#worx-landroid)
* [HTTP binding based control](#HTTP-binding-based-control)

## Worx Landroid
Worx - a company to manufacture garden and homeworking tools - is also offering a range of 'Landroid' robot lawn mowers.
Some of the current models (M1000i a.k.a. WG796E.1 and M1500i a.k.a. WG797E) introduced end of 2014 feature onboard WiFi and app control. As the app is essentially just using HTTP and JavaScript, we can reuse this interface to integrate with openHAB.
**NOTE**: there is no official API. This solution is based on reverse engineering the app's access to a WG796E.1 running software versions 1.98 ("Hypericum 3.1"). Latest version it was verified to work with currently is 2.21.
But keep in mind it may change without notice in future mower OS versions.

Here's a [overview of the 2017 models](http://www.roboter-forum.com/showthread.php?18592-Spezifikationen-der-WORX-Landroid-2017-Modelle). Seems they now all have WiFi, but that's preliminary information.


## MQTT based control
For a MQTT and Python based solution, check out [this project](https://github.com/trieb/worx-landroid).


## HTTP binding based control
Use the openHAB HTTP binding to directly query and control Landroids.

(replace IP and use your Landroid PIN instead of "1234")

    String Landroid_status "Landroid Status [%s]" <garden> (Rasenmaeher) { http="<[http://admin:1234@10.0.0.1/jsondata.cgi:5000:REGEX(.*\"state\":\"(.*?)\",.*)]" }
    String Landroid_Command "Landroid action" <garden> (Rasenmaeher)

If you call the URL, output shows a number of parameters, `state` being just one of them.
Feel free to pick those of interest and use a copy of the `Landroid_status` item with an appropriately modifed regex.

in sitemap:

    Text item=Landroid_status label="Landroid status [%s]"
    Switch item=Landroid_Command mappings=[11="Start", 12="Stop"]

put up a rule:

    rule "Landroid command"
    when
        Item Landroid_Command received command
    then
        // Commands
        // 11 = start
        // 12 = stop (& return to base)
        // 13 = charging complete
        // 14 = manual stop
        // 15 = going home

        // NOTE: insert your PIN and IP here
        val String URL = "http://admin:1234@10.0.0.1/jsondata.cgi"

        var String jsondata = 'data=[[\"settaggi\",' + Landroid_Command.state.toString + ',1]]'


        logInfo("rules", "Sending command " + Landroid_Command.state + " to Landroid.")

        sendHttpPostRequest(URL, "application/x-www-form-urlencoded", jsondata)
    end
