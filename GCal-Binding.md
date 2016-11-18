
> Starting from version **1.9.0** binding has been refreshed to support Google OAuth2 and Google calendar API v3. Only this version is currently supported by Google. Older version does not work.

# Documentation of the Google Calendar IO-Bundle

## Introduction

If you want to administer events in Google Calendar that will be executed by openHAB this bundle will do the job. See the following sections on how to configure the necessary data and how to obtain the GCal URL. 

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

or

    > script

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

or

    > Light_Garden.sendCommand(ON)

## Obtain the credentials

Before you can integrate OpenHAB with Google calendar you must have a Google API Console project.
* Login to [https://console.developers.google.com](https://console.developers.google.com)
* From the project drop-down, select an existing project  , or create a new one by selecting **Create a new project**.
* In the sidebar under "API Manager", select Credentials, then select the OAuth consent screen tab.
* Choose an **Email Address**, specify a **Product Name**, and press Save.
* In the Credentials tab, select the Create credentials drop-down list, and choose **OAuth client ID**.
* Under **Application type**, select **Other**. 
* Put **Name** and press the **Create** button.
* Copy **client id** and **client secret**

The client id, client secret and calendar name has to be configured in your openhab.cfg. 

- fill in gcal:client_id, gcal:client_secret in openhab.cfg
- login in to https://www.google.com/calendar/
- find calendar name you want to use (under "My calendars")
- fill in gcal:calendar_name in openhab.cfg. If you want to use your primary calendar just put keyword "primary"

After first start you need to authorize openHAB to allow use your calendar. Follow openHAB console for instruction:

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
- content: `> if (PresenceSimulation.state == ON) sendCommand(<itemname>,<value>)`

To make use of the Presence Simulation you have to walk through these configuration steps:

- make sure that you are using the correct openHAB release (at least 1.9.0-SNAPSHOT)
- configure the gcal-persistence bundle by adding the appropriate configuration in your `openhab.cfg`. All entries start with `gcal-persistence`. You must add only calendar_name. All other credentials are shared from gcal.
- make sure your items file contains items that belong to the group `PresenceSimulationGroup` - if you would like to change the group name change it at `gcal.persist`.
- make sure your items file contains an item called `PresenceSimulation` which is referred by the scripts executed at a certain point in time - if you would like to change the group name please change the parameter `gcal-persistence:executescript` in your openhab.cfg`.
- make sure the referenced gcal calendar is writeable by the given user (google calendar website)

Note: you also need to configure the gcal-io binding (Google Calendar Configuration in 'openhab.cfg') to be able to read the entries from the calendar and act on it!

To activate the Presence Simulation simply set `PresenceSimulation` to `ON` and the already downloaded events are being executed. Your smartHome behaves like you did 14 days ago.

A sample `gcal.persist` file looks like this:

    Strategies {
    	default = everyChange
    }
    
    Items {
    	PresenceSimulationGroup* : strategy = everyChange
    }

## Solving gcal persistence errors:
To solve any issues with any binding, increase the logging. For gcal, add these lines to your 'logback.xml'

    <logger name="org.openhab.persistence.gcal" level="TRACE" />
    <logger name="org.openhab.io.gcal" level="TRACE" />

* "GCal PresenceSimulation Service isn't initialized properly! No entries will be uploaded to your Google Calendar"

    The persistence configuration is not correct; username, password and url are required.
    Configuration entries must be prefixed by `gcal-persistence:`.

* "creating a new calendar entry throws an exception: Forbidden"

    This can have several causes:
    * The client_id/client_secret might not be correct

    * The calendar name is not correct.

    * If your not using your own calendar, make sure the sharing settings are correct and the user has sufficient rights to create calendar entries.
