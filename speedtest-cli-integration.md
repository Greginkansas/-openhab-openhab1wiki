Below you'll find a complete solution on integrating the internet connection bandwidth speed test from [Speedtest.net](http://www.speedtest.net) in your OpenHAB setup. This is accomplished by using the [`speedtest-cli`](https://github.com/sivel/speedtest-cli) script, which you have to provide on your system.

**Functionality:**
  * execute speed test once or twice a day, every hour or upon command
  * show summarized results on the sitemap
  * show all results on an extra page after click
  * a button is defined to start the speedtest, either through the sitemap or your own rules

<img src="https://community-openhab-org.s3-eu-central-1.amazonaws.com/original/2X/2/2b3ee536c3026d68191802329246b3bca6a7dd3f.png" width="300">

## Installation and Test
### Linux
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

### Windows
You can find an unofficial speedtest-cli for Windows here: https://github.com/zpeters/speedtest

Place the executable on your harddrive, e.g. "`C:\openHAB\speedtest.exe`"

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

### Icons
All icons in one archive: [Download](https://raw.githubusercontent.com/ThomDietrich/openhab-config/master/additional-images/speedtest-icons.rar)

Extract the archive and move all icons to the openHAB images folder under `${openhab_home}/webapps/images`.

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

### Rule (Linux)
```Xtend
import org.openhab.core.library.types.DateTimeType
  
rule "Speedtest init"
when
	System started
then {
	if (SpeedtestSummary.state == Uninitialized || SpeedtestSummary.state == "") SpeedtestSummary.postUpdate("⁉ (unbekannt)")
}
end

rule "Speedtest"
when
  	//Time cron "0 0 5 * * ?" or
  	//Time cron "0 0 13 * * ?"
  	Time cron "0 0 * * * ?" or
  	Item SpeedtestRerun received command ON
then {
	logInfo("RULE", "--> speedtest executed...")
	SpeedtestRunning.postUpdate("Messung läuft...")
	
	// update timestamp for last execution
	SpeedtestResultDate.postUpdate(new DateTimeType())
	
	// execute the script, you may have to change the path depending on your system
	var String speedtestCliOutput = executeCommandLine("/usr/local/bin/speedtest-cli@@--simple", 120*1000)
	
	// for debugging:
	//var String speedtestCliOutput = "Ping: 43.32 ms\nDownload: 21.64 Mbit/s\nUpload: 4.27 Mbit/s"
	//logInfo("RULE", "--> speedtest output:\n" + speedtestCliOutput + "\n\n")
	
	SpeedtestRunning.postUpdate("Datenauswertung...")
	
	// starts off with a fairly simple error check, should be enough to catch all problems I can think of
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
		logInfo("RULE", "--> speedtest finished.")
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

That's it. If you have problems, just activate the logging lines and have a look in your `openhab.log` to get an idea of what's going on.


### Rule (Windows)
The following changes are needed compared to the Linux rule above:

```Xtend
...
var String speedtestCliOutput = executeCommandLine("c:\\openHAB\\speedtest.exe@@--report", 120*1000)
...
if (speedtestCliOutput.startsWith("201")) {        
    var String[] results = speedtestCliOutput.split("\\|")
    var float ping = new java.lang.Float(results.get(3))
    var float down = new java.lang.Float(results.get(4))
    var float up   = new java.lang.Float(results.get(5))
    SpeedtestResultPing.postUpdate(ping)
    SpeedtestResultDown.postUpdate(down/1024)
    SpeedtestResultUp.postUpdate(up/1024)
...
```

This idea was originally discussed and questions should be asked at https://community.openhab.org/t/speedtest-cli-integration/7611