This page is work-in-progress to summarize the findings of the following community discussion for users hosting openHAB on a M$-Windows operating system [[1](https://community.openhab.org/t/googletts-responding-with-503-error-even-after-the-url-fix/3385)].

***
* [Environment Requirements](#environment-requirements)
* [REST-TTS service](#rest-tts-service)
* [Configuration](#configuration)
  * [REST-TTS](#rest-tts)
  * [Operating System](#operating-system)
  * [OpenHAB](#openhab)
* [References](#references)

***

# Environment Requirements
To verify if your system provides decent text-to-speech ability I recommend to download "Balabolka" [[2](http://portableapps.com/apps/accessibility/balabolka-portable)]. Verify that a SAPI 5 voice is available for your language and that it's quality is decent enough.

![Balabolka - SAPI 5](https://dl.dropboxusercontent.com/u/1781347/wiki/Balabolka-SAPI5.png)

If the voice quality is not sufficient, or the desired language is missing you may consider a commercial SAPI 5 voice. Only use trial versions until your openHAB TTS is working end-to-end, as I can not know if the solution is compatible with all voices.

To use this with squeezebox binding you need a recent 1.8.0 snapshot (or newer) build of the squeezebox add-ons; available from cloudbees [[4](https://openhab.ci.cloudbees.com/job/openHAB)].

# REST-TTS service
One way to use OS-TTS with openHAB (e.g. Squeezebox) is to provide a REST API that will convert a text string to an .mp3 file with the respective spoken text. The software available from [[3](https://dl.dropboxusercontent.com/u/1781347/RESTTTS-2016-01-01.001.zip)] provides such an API.

Extract the downloaded .zip package. This will provide you the source code of the utility (can be compiled using M$-VS 2013 community edition) as well as some pre-compiled ready to use files (.\RESTTTS\RESTTTS\bin\Release).
# Configuration
## REST-TTS
To keep things as simple as possible the tool is a self hosted REST service. This requires elevated rights on your system. Therefore either compile the binaries yourself (after review of the code), or if you use a pre-compiled version make sure you scan it with a recent anti-virus software.

Right-click the "RESTTTS.exe" and execute it as administrator; you should see something similar to this:
![REST TTS with default configuration](https://dl.dropboxusercontent.com/u/1781347/wiki/2016-01-03%2016_03_48-_RESTTTS_RESTTTS_bin_Rele.png)

Enter the following URL in a new browser window (or tab): "http://127.0.0.1:8089/Service/TTS?text=Test"; an .mp3 file should be received by the browser. Once that 1st test works ok close the REST service again (just select the windows and press enter).

Now open "RESTTTS.exe.config" and replace the loopback address with your actual IP (e.g. 192.168.1.100). If you prefer a different voice, enter the voice id as well (e.g. "TTS_MS_EN-US_ZIRA_11.0"):

![RESTTTS.exe.config](https://dl.dropboxusercontent.com/u/1781347/wiki/2016-01-03%2016_12_06-Starten.png) 

Start the service after the configuration change and check that the new settings were applied.
 
## Operating System
By default the firewall of the operating system will allow external systems to access the service from other computers in your network. Therefore you need to configure an exception that you allow incoming request to the configured port (e.g. 8089) from the local network.

Verify that this works correctly with a browser from a different computer in you network using the configured network address and port (e.g. http://192.168.1.100:8089/Service/TTS?text=Test).

## OpenHAB

Now you can change the openHAB configuration to instruct squeezebox to use this service:

```
squeeze:ttsurl               = http://192.168.10.100:8089/Service/TTS?text=%s
squeeze:ttsmaxsentencelength = 4096
```

***
# References
* [1]: [GoogleTTS responding with 503 error](https://community.openhab.org/t/googletts-responding-with-503-error-even-after-the-url-fix/3385)
* [2]: [Balabolka Portable](portableapps.com/apps/accessibility/balabolka-portable)
* [3]: [REST-TTS](https://dl.dropboxusercontent.com/u/1781347/RESTTTS-2016-01-01.001.zip)
* [4]: [openHAB nightly builds](https://openhab.ci.cloudbees.com/job/openHAB)