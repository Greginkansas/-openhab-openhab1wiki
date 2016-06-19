openHAB Android is an open-source application providing openHAB user interface on Android platform. It is an integral part of openHAB project.

<img src="http://www.openhab.org/assets/images/ui/android-screens.png">

# Basic information on Android app

Please use [openHAB Android issues](https://github.com/openhab/openhab.android/issues) to submit bugs, feature requests and ideas on how to improve HABDroid.

**Connecting to openHAB**

Android app uses Bonjour/mDNS service discovery to find openHAB in local network. During startup Android app detects the type of active network connection. If WiFi/Ethernet is used for network connection Android app tries to discover openHAB.
If Android app is unable to discover openHAB on local network, it assumes this is not a home network and uses Remote URL from application settings to connect to openHAB. It goes the same way if mobile network connection is detected. All settings, including username, password and remote URL can be configured through 'Settings' menu which is available at any point by pressing phone 'Menu' button and selecting 'Settings' from popup menu. You can also configure a static URL (openHAB URL) in Settings and openHAB will never try to discover and will connect to this URL during startup. If local URL is not reachable app will failover to remote URL.
When you connect to openHAB for the first time Android app will ask you to select sitemap from the list of available sitemaps. It will remember your decision and open this sitemap automatically next time you launch the app. You can switch sitemap anytime using 'Select sitemap' from application menu.

**Security**

We strongly recommend to use HTTPS, though HTTP is also available.

If username and password are configured Android app will automatically use them for authenticating http requests to openHAB. If openHAB doesn't have any security configured Android app will connect to openHAB anyway.

We strongly recommend to switch on user authentication in openHAB when enabling remote connection over the Internet!

**User experience**

Android app is currently designed for mobile phones and tablets running Android 4.x.

Android app supports all openHAB sitemap widgets.

Android app supports using NFC tags for your home automation!
You can write your sitemap page to NFC tag through application menu -> Write NFC tag. The current open sitemap page will be written to the tag. To open the page just touch the tag with your Android device, even if Android app is not running.
You can also write an item action to NFC tag. Long press the item on a sitemap page (Switch, Rollershutter and Selection items are currently supported), select an action you would like to be performed with this item when touching the tag, then write the tag. To perform the action just touch the tag with your Android device, even if Android app is not running.

Android app supports 2 different color themes, HOLO.Dark (which is default black one) and HOLO.Light (the white one). The theme can be set through Settings menu.

Android app supports disabling screen timeout timer while Android app is running. This is useful for those who use Android device as a wall control panel. This can be set through Settings menu.

Fontsize in charts. To increase the size of the font in charts in the app you need to change the scale setting in the .cfg file.
This should make the text readable in your phone:
chart:scale=0.5


**openHAB version support**

Android app is compatible with openHAB version starting from 1.0.0.

# Using application

Please install release version of Android app from Google Play store:

[![HABDroid](http://www.androiddocs.com/images/brand/en_generic_rgb_wo_45.png)](https://play.google.com/store/apps/details?id=org.openhab.habdroid)

You can always find current development build on [openHAB Cloudbees page](https://openhab.ci.cloudbees.com/job/HABDroid/)
