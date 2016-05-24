openHAB iOS is an open-source application providing openHAB user interface on iOS platform. It is an integral part of openHAB project.

<img src="http://www.openhab.org/images/ui/ios-screens.png">

# Basic information on iOS app

Please use [openHAB iOS issues](https://github.com/openhab/openhab.ios/issues) to submit bugs, feature requests and ideas on how to improve the iOS native app.

**Connecting to openHAB**

iOS app uses Bonjour/mDNS service discovery to find openHAB in local network. During startup iOS app detects the type of active network connection. If WiFi is used for network connection iOS app tries to discover openHAB.
If iOS app is unable to discover openHAB on local network, it assumes this is not a home network and uses Remote URL from application settings to connect to openHAB. It goes the same way if mobile network connection is detected. All settings, including username, password and remote URL can be configured through 'Settings' menu which is available at any point by clicking 'Settings' on the top right corner of the screen. You can also configure a static URL (openHAB URL) in Settings and openHAB will never try to discover and will connect to this URL during startup. If local URL is not reachable app will failover to remote URL.
When you connect to openHAB for the first time iOS app will ask you to select sitemap from the list of available sitemaps. It will remember your decision and open this sitemap automatically next time you launch the app. You can switch sitemap anytime using 'Select sitemap' from settings menu.

The Local URL should be entered like this:
```
https://192.x.x.x:8443
```
Adding `/openhab.app` or more is not necessary and will cause a failed connection.

**Security**

We strongly recommend to use HTTPS, though HTTP is also available.

If username and password are configured iOS app will automatically use them for authenticating http requests to openHAB. If openHAB doesn't have any security configured iOS app will connect to openHAB anyway.

We strongly recommend to switch on user authentication in openHAB when enabling remote connection over the Internet!

**User experience**

iOS app is currently designed for mobile phones and tablets running iOS 7.

iOS app supports all openHAB sitemap widgets.

iOS app supports disabling screen timeout timer while iOS app is running. This is useful for those who use iOS device as a wall control panel. This can be set through Settings menu.

**openHAB version support**

iOS app is compatible with openHAB version starting from 1.0.0.

# Using application

Please install release version of iOS app from the App Store:

[![iOSApp](http://raw.github.com/wiki/openhab/openhab/images/app-store-badges.png)](http://itunes.apple.com/us/app/openhab/id492054521?mt=8)
