# Google Calendar IO-Bundle

> Starting from version **1.9.0** binding has been refreshed to support Google OAuth2 and Google Calendar API v3. Only this version is currently supported by Google; the older versions do not work.

## Introduction

If you want to defined events in Google Calendar that will be executed by openHAB this bundle will do the job. See the following sections on how to configure the necessary data and how to obtain the GCal URL. 

## Calendar Event Configuration

The event title can be anything and the event description will have the commands to execute.

The format of Calendar event description is simple and looks like this:

    start {
      send|update <item> <state>
    }
    end {
      send|update <item> <state>
    }

or just

    send|update <item> <state>

The commands in the `start` section will be executed at the event start time and the `end` section at the event end time. If these sections are not present, the commands will be executed at the event start time.

As a result, your lines in a Calendar event might look like this:

    start {
      send Light_Garden ON
      send Pump_Garden ON
    }
    end {
      send Light_Garden OFF
      send Pump_Garden OFF
    }

or just

    send Light_Garden ON
    send Pump_Garden ON


## Obtain the Credentials

Before you can integrate OpenHAB with Google Calendar you must have a Google API Console project.

* Login to [https://console.developers.google.com](https://console.developers.google.com)
* From the project drop-down, select an existing project  , or create a new one by selecting **Create a new project**.
* In the sidebar under "API Manager", select Credentials, then select the OAuth consent screen tab.
* Choose an **Email Address**, specify a **Product Name**, and press Save.
* In the Credentials tab, select the Create credentials drop-down list, and choose **OAuth client ID**.
* Under **Application type**, select **Other**. 
* Put **Name** and press the **Create** button.
* Copy **client id** and **client secret**

### Service Configuration

gcal.cfg (openHAB 2+)
```
############################### GCal configuration ################################
#
# Before using GCal, you need to have a Google API
# Console project. The Wiki describes in detail the steps necessary to set 
# up your Google API Console project, as well as how to obtain the credentials 
# necessary to complete the information in this file. Once the project is created, 
# and you've completed the steps described in the Wiki, you need to copy 
# the "Client ID" and "Client secret" from the Credentials page on 
# console.developers.google.com
#
#
# Copied from the "Client ID" field on the Credentials page. (required)
#client_id=

# Copied from the "Client secret" field on the Credentials page (required)
#client_secret=

# This is the name you gave to your Google Calendar, or the word 'primary' if you
# want to use your default Google calendar.  GCal will download calendar events 
# from this calendar (required)
#calendar_name=

# The filter criteria by which calendar events are searched. The Google Calendar 
# API will do a text search to find calendar events that match the supplied terms. 
# All calendar event fields are searched, except for extended properties (optional)
#filter=

# Refresh interval (in milliseconds) is the frequency with which the
# Google calendar will be checked for calendar events (optional, defaults 
# to 900000 [15 minutes], requires restart)
#refresh=
```

openhab.cfg fragment (openHAB 1.x):
```
####################### Google Calendar (GCal) configuration ##########################
#
# Before using GCal, you need to have a Google API
# Console project. The Wiki describes in detail the steps necessary to set 
# up your Google API Console project, as well as how to obtain the credentials 
# necessary to complete the information in this file. Once the project is created, 
# and you've completed the steps described in the Wiki, you need to copy 
# the "Client ID" and "Client secret" from the Credentials page on 
# console.developers.google.com
#
# Copied from the "Client ID" field on the Credentials page (required)
#gcal:client_id=

# Copied from the "Client secret" field on the Credentials page (required)
#gcal:client_secret=

# This is the name you gave to your Google Calendar, or the word 'primary' (without
# quotes) if you want to use your default Google calendar.  GCal will download 
# calendar events from this calendar (required)
#gcal:calendar_name=

# The filter criteria by which calendar events are searched. The Google Calendar 
# API will do a text search to find calendar events that match the supplied terms. 
# All calendar event fields are searched, except for extended properties (optional)
#gcal:filter=

# Refresh interval (in milliseconds) is the frequency with which the
# Google calendar will be checked for calendar events (optional, defaults 
# to 900000 [15 minutes])
#gcal:refresh=
```

After first start you need to authorize openHAB to allow use your calendar. Follow openHAB log for instructions.  At the `openhab>` prompt, enter `log:tail` (openHAB 2+).  At the shell prompt enter `tail -f /path/to/your/openhab.log`.

     [INFO ] [g.internal.GCalEventDownloader] -################################################################################################
     [INFO ] [g.internal.GCalEventDownloader] - # Google-Integration: U S E R   I N T E R A C T I O N   R E Q U I R E D !!
     [INFO ] [g.internal.GCalEventDownloader] - # 1. Open URL 'https://www.google.com/device'
     [INFO ] [g.internal.GCalEventDownloader] - # 2. Type provided code ZPWT-UVXXS 
     [INFO ] [g.internal.GCalEventDownloader] - # 3. Grant openHAB access to your Google calendar
     [INFO ] [g.internal.GCalEventDownloader] - # 4. openHAB will automatically detect the permiossions and complete the authentication process
     [INFO ] [g.internal.GCalEventDownloader] - # NOTE: You will only have 1800 mins before openHAB gives up waiting for the access!!!
     [INFO ] [g.internal.GCalEventDownloader] -################################################################################################


# Presence Simulation

The GCal persistence bundle can be used to realize a simple but effective Presence Simulation feature (thanks Ralf for providing the concept). Every single change of an item that belongs to a certain group is posted as new calendar entry in the future. By default each entry is posted with an offset of 14 days (If you'd like to change the offset please change the parameter `gcal-persistence:offset` in your `openhab.cfg`). Each calendar entry looks like the following:

- title: `[PresenceSimulation] <itemname>`
- content: `send <itemname> <value>`

To make use of the Presence Simulation you have to walk through these configuration steps:

- make sure that you are using the correct openHAB release (at least 1.9.0-SNAPSHOT)
- configure the gcal-persistence bundle by adding the appropriate configuration in your `openhab.cfg`. All entries start with `gcal-persistence`. You must add only calendar_name. If you want to use your primary calendar just put the keyword "primary" (without quotes). All other credentials are shared from gcal.
- make sure your items file contains items that belong to the group `G_PresenceSimulation` - if you would like to change the group name change it at `gcal.persist`.
- make sure your items file contains an item called `PresenceSimulation` which is referred by the scripts executed at a certain point in time - if you would like to change the group name please change the parameter `gcal-persistence:executescript` in your openhab.cfg`.
- make sure the referenced gcal calendar is writeable by the given user (google calendar website)

Note: you also need to configure the gcal-io binding (Google Calendar Configuration in 'openhab.cfg') to be able to read the entries from the calendar and act on it!

To activate the Presence Simulation simply set `PresenceSimulation` to `ON` and the already downloaded events are being executed. Your smartHome behaves like you did 14 days ago.

A sample `gcal.persist` file looks like this:

    Strategies {
    	default = everyChange
    }
    
    Items {
    	G_PresenceSimulation* : strategy = everyChange
    }

## Solving gcal persistence errors

To solve any issues with any binding, increase the logging. For gcal, add these lines to your 'logback.xml' (openHAB 1.x):

    <logger name="org.openhab.persistence.gcal" level="TRACE" />
    <logger name="org.openhab.io.gcal" level="TRACE" />

Under openHAB 2+, at the `openhab>` prompt you can enter:

```
log:set TRACE org.openhab.persistence.gcal
log:set TRACE org.openhab.io.gcal
```

You can later return them to `DEFAULT` level.

* "GCal PresenceSimulation Service isn't initialized properly! No entries will be uploaded to your Google Calendar"

    The persistence configuration is not correct; check that the client_id and client_secret are correct in the gcal configuration.
    Configuration entries must be prefixed by `gcal-persistence:`.

* "creating a new calendar entry throws an exception: Forbidden"

    This can have several causes:
    * The client_id/client_secret might not be correct

    * The calendar name is not correct.

    * If your not using your own calendar, make sure the sharing settings are correct and the user has sufficient rights to create calendar entries.
