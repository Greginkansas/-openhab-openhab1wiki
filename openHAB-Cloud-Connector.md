The openHAB Cloud Connector allows connecting the local openHAB runtime to a remote [openHAB Cloud](https://github.com/openhab/openhab-cloud/blob/master/README.md) instance, such as [myopenHAB.org](https://www.myopenHAB.org), which is an instance of the
openHAB Cloud service hosted by the [openHAB Foundation](http://www.openhabfoundation.org/).

## Features

The openHAB Cloud service (and thus the connector to it) is useful for different use cases:

* It allows remote access to local openHAB instances without having to expose ports to the Internet or to require a complex VPN setup.
* It serves as a connector to Google Cloud Messaging (GCM) and Apple Push Notifications (APN) for pushing notifications to mobile phone apps.
* It brings integration possibilities with services that require an OAuth2 authentication against a web server, such as IFTTT or Amazon Alexa Skills.

## UUID and Secret

To authenticate with the openHAB Cloud your local openHAB runtime generates two values, which need to be entered in your account settings of the openHAB Cloud service. The first one is a unique identifier, which allows to identify your runtime. One can think of it as something similar like a username for the cloud authentication. The second one is a random secret key which serves as a password. Both values are written to the local file system. If you loose these files for some reason, openHAB will automatically generates new ones. You will then have to reconfigure UUID and secret in the openHAB Cloud service under the _My account_ section.

Location of UUID and Secret:

|File | Location | APT Installation |
|-----|----------------------|------------------|
|UUID | ${openhab.home}/webapps/static/uuid        | /usr/share/openhab/webapps/static/uuid |
|Secret | ${openhab.home}/webapps/static/secret | /usr/share/openhab/webapps/static/secret |

## Installation

This bundle will be part of the upcoming openHAB 1.9 add-ons release. For the time being, it is [available for download on Bintray](https://dl.bintray.com/openhab/bin/org.openhab.io.openhabcloud_1.9.0.201612192331.jar). It can be used on any openHAB 1.8 runtime by putting it in the addons folder. 

_Note:_ You require at least Java 1.8.0_101 for the bundle to work properly, if you are using a server with Let's Encrypt certificates (which is the case for myopenhab.org).

## Configuration

You can configure the connector by adding these entries to your `configuration/openhab.cfg`:

```
############################## openHAB Cloud Connector #############################

# The URL of the openHAB Cloud service to connect to.
# Optional, default is set to the service offered by the openHAB Foundation
# (https://myopenhab.org/)
#openhabcloud:baseURL=

# Local port that the openHAB runtime is available through HTTP.
# Optional, default is 8080
#openhabcloud:localPort=

# Defines the mode in which you want to operate the connector.
# Possible values are:
# - notification: Only push notifications are enabled, no remote access is allowed.
# - remote: Push notifications and remote access are enabled.
# Optional, default is 'remote'.
#openhabcloud:mode=

# A comma-separated list of items to be exposed to external services like IFTTT. 
# Events of those items are pushed to the openHAB Cloud and commands received for
# these items from the openHAB Cloud service are accepted and sent to the local bus.
# Optional, default is an empty list.
#openhabcloud:expose=
```

Note that in contrast to the old my.openHAB bundle, there is no support for persistence configurations. The "expose" configuration is the replacement for that.

E.g.:
```
openhabcloud:expose=Window_Livingroom, Window_Sleepingroom
```
Wildcards an goups are not supported.