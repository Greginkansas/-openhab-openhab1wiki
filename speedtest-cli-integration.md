Below you'll find a complete solution on integrating the internet connection bandwidth speed test from [Speedtest.net](http://www.speedtest.net) in your OpenHAB setup. This is accomplished by using the [`speedtest-cli`](https://github.com/sivel/speedtest-cli) script, which you have to provide on your system.

**Functionality:**
  * execute speed test once or twice a day, every hour or upon command
  * show summarized results on the sitemap
  * show all results on an extra page after click
  * a button is defined to start the speedtest, either through the sitemap or your own rules

## Installation and Test

In your linux console (installation may differ from system to system):
```bash
# You'll need the exec binding
sudo apt-get install openhab-addon-binding-exec
#and easy_install
sudo apt-get install python-setuptools
# Installation of speedtest-cli
sudo easy_install speedtest-cli
# Testing
speedtest-cli
speedtest-cli --simple
```
Please make sure the script is functioning before you continue.

## OpenHAB files

### Items
```php
String   SpeedtestSummary    "Speedtest [%s]"                <"network-icon">
Number   SpeedtestResultPing "Ping [%.3f ms]"                <"next-5-icon">
Number   SpeedtestResultDown "Downlink [%.2f Mbit/s]"        <"download-icon">
Number   SpeedtestResultUp   "Uplink [%.2f Mbit/s]"          <"upload-icon">
String   SpeedtestRunning    "Speedtest running ... [%s]"    <"new-icon">
Switch   SpeedtestRerun      "Manuell starten"               <"reload-2-icon">
DateTime SpeedtestResultDate "Zuletzt ausgeführt [%1$td.%1$tm.%1$tY, %1$tH:%1$tM Uhr]" <"problem-4-icon">
```
Icons source (free for non-commercial use): http://www.iconarchive.com/show/polygon-icons-by-graphicloads.html

### Sitemap
```php
...
Text item=SpeedtestSummary {
  Frame label="Ergebnisse" {
    Text item=SpeedtestResultDown
    Text item=SpeedtestResultUp
    Text item=SpeedtestResultPing
  }
  Frame label="Steuerung" {
    Text item=SpeedtestResultDate
    Text item=SpeedtestRunning label="Speedtest [%s]" visibility=[SpeedtestRunning != "-"]
    Switch item=SpeedtestRerun mappings=[ON="Start"]
  }
  Frame label="Statistik" {
    Text label="..." icon="analytics-8-icon"
  }
}
...
```

### Rule
```Xtend
rule "Speedtest"
when
    //TODO: choose one timed model (5:00 + 13:00 OR hourly)
    //Time cron "0 0 5 * * ?" or Time cron "0 0 13 * * ?" or
    Time cron "0 0 * * * ?" or
    // additionally scan on button pressed
    Item SpeedtestRerun received command ON
then {
    //logInfo("RULE", "--> speedtest executed...")
    SpeedtestRunning.postUpdate("Messung läuft...")
    
    // update timestamp for last execution
    postUpdate(SpeedtestResultDate, new DateTimeType(now.toCalendar(null)))
    
    // execute the script, you may have to change the path depending on your system
    var String speedtestCliOutput = executeCommandLine("/usr/local/bin/speedtest-cli@@--simple", 120*1000)
    
    // for debugging:
    //var String speedtestCliOutput = "Ping: 43.32 ms\nDownload: 21.64 Mbit/s\nUpload: 4.27 Mbit/s"
    //logInfo("RULE", "--> speedtest output:\n" + speedtestCliOutput + "\n\n")
    
    SpeedtestRunning.postUpdate("Datenauswertung...")
    
    // starts off with a fairly simple error check, should be enough to catch most obvious problems
    if (speedtestCliOutput.startsWith("Ping") && speedtestCliOutput.endsWith("Mbit/s")) {        
        var String[] results = speedtestCliOutput.split("\\r?\\n")
        var float ping = new java.lang.Float(results.get(0).split(" ").get(1))
        var float down = new java.lang.Float(results.get(1).split(" ").get(1))
        var float up   = new java.lang.Float(results.get(2).split(" ").get(1))
        SpeedtestResultPing.postUpdate(ping)
        SpeedtestResultDown.postUpdate(down)
        SpeedtestResultUp.postUpdate(up)
        SpeedtestSummary.postUpdate(String::format("ᐁ  %.1f Mbit/s  ᐃ %.1f Mbit/s (%.0f ms)", down, up, ping))
        SpeedtestRunning.postUpdate("-")
        //logInfo("RULE", "--> speedtest finished.")
    } else {
        SpeedtestResultPing.postUpdate(0)
        SpeedtestResultDown.postUpdate(0)
        SpeedtestResultUp.postUpdate(0)
        SpeedtestSummary.postUpdate("(unbekannt)")
        SpeedtestRunning.postUpdate("Fehler bei der Ausführung")
        logError("RULE", "--> speedtest failed. Output:\n" + speedtestCliOutput + "\n\n")
    }
    SpeedtestRerun.postUpdate(OFF)
}
end
```

That's it. If you have a problem with the script, activate the logging lines and have a look in your `openhab.log`.

This idea was originally discussed and questions should be asked at https://community.openhab.org/t/speedtest-cli-integration/7611