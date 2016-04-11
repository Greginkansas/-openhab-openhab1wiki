WIP to be completed
This Section describes how to use AT Heatmiser Wifi Project together with JSONPath and execute commands and actions to completely control an Heatmiser Thermostat.

This is not a binding but is a a set of items/sitemap/rules file that use per Perl development by A. Thoukydides that can be found at https://github.com/thoukydides/heatmiser-wifi

As a first prerequisite please follow the instructions found at https://github.com/thoukydides/heatmiser-wifi/wiki
As far as Perl is concerned I installed the full CPan not CPanminus (It might work also with cpanminus).

You just need the perl script interacting with Json for the sake of OpenHab Integration, so Installation of the WebServer and MySql is optional, but once you are there it is a nice addon for your olf Heatmiser...

Be advised that installation of the prerequisite will take a rather long time. 

Prepare a directory on your device at the following path (tobeinserted).
This folder will contain simple bash oneliners calling the Perl command with proper parameters.
The scripts do the following:
Onleliner name                                Function

Finally:
integrate the following to your sitemap/items/rules.

Please adapt the icon names to the ones you want to use.

Please bear in mind that this is still largely WIP but is published with the intent to finish it with help of the community as I have not much  time to dedicate to it. So... feel free to complete/make it better.

Unix Bash Oneliners Scripts:

Sitemap:

Items:

Rules:





