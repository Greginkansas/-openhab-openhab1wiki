# Using Charts in openHAB

# Introduction

With [[Persistence|Persistence]] being available in openHAB, one frequent requirement is to visualize time series of values in the UI. openHAB supports the easy definition and creation of such charts.

# Available Solutions

Currently only the server-side chart creation as PNG through rrd4j is implemented. 

There are other potential ways to integrate charts in the UI: They could be created on the server and provided to the UI as PNG or SVG files. They could also be created as HTML5 through Javascript (see e.g. [Sencha Charts](http://www.sencha.com/products/touch/charts) or [Google Chart Tool](https://developers.google.com/chart/)), where the data would be retrieved from the openHAB server through a REST API.


## Using RRD4j Charts

To use the rrd4j charting solution, you must have installed the addon bundle org.openhab.persistence.rrd4j.
The items that you want to show on the graphs must be persisted through the rrd4j persistence service _once a minute_, see the [[Persistence documentation|Persistence]] on how to set this up.

__WARNING: The RRD chart needs data to be stored at least once per minute, otherwise it will not show  data.__

Charts are not created in a regular interval, but on-the-fly upon access, which means that they are always up-to-date. To access the charts, the org.openhab.persistence.rrd4j bundle registers a servlet that can be accessed at `http://<server>:<port>/rrdchart.png`.

What kind of data should be displayed is configured through HTTP parameters. The servlet accepts the following ones:
- w: width in pixels of image to generate - this parameter is optional and defaults to 480
- h: height in pixels of image to generate - this parameter is optional and defaults to 240
- period: the time span for the x-axis. Value can be h,4h,8h,12h,D,3D,W,2W,M,2M,4M,Y - this parameter is optional and defaults to "D", i.e. showing the last 24 hours
- items: A comma separated list of item names to display - mandatory, unless "groups" parameter is provided
- groups: A comma separated list of group names, whose members should be displayed - mandatory, unless "items" parameter is provided

A valid request hence could look like this:

    http://localhost:8080/rrdchart.png?items=Weather_Temperature,Weather_Temp_Max,Weather_Temp_Min&period=W

The created chart will be automatically formatted with some sensible defaults. There is currently no way to change e.g. the y-axis range, the line colors, the legend, etc. A flexible configuration solution that allows full control of all rrd tool features is planned for later.

## Using default Charts

Default charts are accessible via ```http://localhost:8080/chart``` using same parameter as above. Example:

    http://localhost:8080/chart?groups=Heizung&period=W&random=1&h=800&w=1200