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

# REST-TTS service
One way to use OS-TTS with openHAB (e.g. Squeezebox) is to provide a REST API that will convert a text string to an .mp3 file with the respective spoken text. The software available from [[3](https://dl.dropboxusercontent.com/u/1781347/RESTTTS-2016-01-01.001.zip)] provides such an API.

Extract the downloaded .zip package. This will provide you the source code of the utility (can be compiled using M$-VS 2013 community edition) as well as some pre-compiled ready to use files (.\RESTTTS\RESTTTS\bin\Release).
# Configuration
## REST-TTS
...

## Operating System
...

## OpenHAB
... 

***
# References
* [1]: [GoogleTTS responding with 503 error](https://community.openhab.org/t/googletts-responding-with-503-error-even-after-the-url-fix/3385)
* [2]: [Balabolka Portable](portableapps.com/apps/accessibility/balabolka-portable)
* [3]: [REST-TTS](https://dl.dropboxusercontent.com/u/1781347/RESTTTS-2016-01-01.001.zip)