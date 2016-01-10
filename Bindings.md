This page describes how openHAB binding can be installed

## Introduction

Bindings are optional packages that can be used to extend functionality of openHAB. By help of bindings openHAB users can e.g. access Asterisk communications software or connect to the KNX Home Automation Bus.

To get an overview, visit the [tagged binding list](http://www.openhab.org/features/supported-technologies.html).

## Installation

openHAB bindings do not need a dedicated installation but three simple steps:

1. Copy binding package to the respective folder
2. Configure basic settings for binding
3. Bind items and rules to the binding

### Step 1: Copy binding package to the respective folder

All openHAB bindings are available as pre-packaged jar-files. 

Please visit [openHAB's download page](http://www.openhab.org/downloads.html) and download the latest version of the addons-bundle. This bundle download contains all available bindings as individual pre-packages files.

Unzip the bundle package and place all binding you want to use in the folder ${openhab_home}/addons.

### Step 2: Configure basic settings for binding

Visit openHAB's WIKI to check which basic configuration settings you can use for the bindings you want to use.

In openhab's configuration file "openhab.cfg" you can set these basic configuration settings.

### Step 3: Bind items and rules to the binding

Visit openHAB's WIKI to check how to bind items, actions and rules to the bindings.

That's it!

### Troubleshooting

Ensure the there are no trailing spaces in the openhab.cfg file when adding basic configuration settings. All entries should begin immediately with no formatting.