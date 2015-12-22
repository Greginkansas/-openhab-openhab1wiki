Documentation of the Snapshot binding bundle for use with CCTV cameras.

## Introduction

This binding is designed to be used in conjuction with cameras that can send snapshot JPG images to the server that OpenHab is running on.  The binding continually monitors directories for new images and fires events when a new image arrives.  This means that you can build rules to do cool things when your cameras detect movement such as ring a door bell, send a notification or sound an alarm.

The binding includes a web servlet which generates html pages and thumbnails so you can conveniently browse your snapshots.

The binding is also capable of capping the number of images in the directory based upon either file count or number of days so you don't run out of disk space.

## Generic Item Binding Configuration

In order to bind an item to a snapshot directory, all you need to do is configure a generic item binding in your items configuration folder like this:

`DateTime cam1 "Cam 1 [%1$td/%1$tm/%1$tY %1$tT %1$tZ]" <calendar> { snapshot="path=/path/to/cam1;maxFiles=50" }`

Note that the type of the item must be `DateTime` because it contains the timestamp of the most recent snapshot.

You can of course have multiple items if you have multiple cameras/directories.

## Debugging

Modify your logback.xml file to debug if needed:

`<logger name="org.openhab.binding.snapshop" level="DEBUG" />`

## Managing Snapshots

When a new snapshot appears then the binding does the following:

* if the option `maxDays` is configured, it deletes any snapshots older than the given value
* if the option `maxFiles` is configured, it sorts the snapshots by creation date, then deletes the oldest files so that the file count for the given folder does not exceed the given value

## Web servlet

The binding has a webservlet with the following features:

* renders a servlet at /snapshot to summarise the folders and their contents
* groups the snapshots based on timestamp (if snapshots are less than a minute apart they are presumed to relate to the same event and so are grouped together). This means it's easy to browse recent 'events'. The snapshot shown in the summary view is the middle snapshot.
* renders a webpage to provide a list of snapshots in a specific group
* uses tokens and template files (I took the concept from the excellent [weather binding](https://github.com/openhab/openhab/wiki/Weather-Binding)) so that you can fully customise the webpages
* can optionally resize snapshots to create thumbs/previews in the webpages
* can render the original snapshot

To use it, create a subdirectory in webapps called `snapshot-resources` and provide template files for the various views.  Please see this [example zip of template files](https://drive.google.com/file/d/0B3zmi0FWXByrT1VzTWJ6N3p0a3M/view?usp=sharing) which should be self explanatory.