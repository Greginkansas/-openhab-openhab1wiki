Shell-scripts to update to a snapshot-build

## Introduction

If you are experimenting with the latest openHAB-snapshots, it can be quite annoying to download all nightly-packages, extract them, update all files and addons and copy/replace the configuration files, every time.


To automate this process there are two shell-scripts for Linux available.

### Script 1

**New Version** (2015/1/18: script supports weather-binding and grafana)

the script [openhab_get_snapshot.sh](http://code.google.com/p/openhab-samples/source/browse/scripts/openhab_get_snapshot.sh?repo=wiki) is a quick-and-dirty-script to download a new openHAB Snapshot to a new folder.

Usage is `openhab_get_snapshot.sh [nnn]` , where nnn is the Build-Number. When nn is not set,  openhab_get_snapshot.sh will download the youngest Build. 
**Since Version 1.5.0 two different Versions are built, so look at the Version!**

All "dynamic" configurations (Which Designer should be downloaded? Is demo needed? What's with GreenT? Want HABmin also? Which addons should be activated?) are set in [config-file](https://code.google.com/p/openhab-samples/source/browse/scripts/getsnap.cfg?repo=wiki). This File is per default stored under `/etc/default/getsnap` but this path can be set in the Script.

The script will create a subdirectory under (per default) `/srv/openhab/` with the name **version-nnn** (e.g. 1.3.0-461), then download all configured packages to a sub-subdirectory `zips/` (at least runtime and addons), unzip the packages, move configured addons and link `configurations/`, `etc/` and `webapps/images/` to `/srv/openhab/`**`subdir`**`/`. The new runtime-path is symlinked to /opt/openhab.

At the end, all that has to be done is

    cd /opt/openhab

    ./start.sh

All addons are unzipped to runtime/addons_inactive, then the configured addons are moved to `runtime/addons/`. So it is easy to activate more addons by moving the addons and reconfigure `/etc/default/getsnap`.

A assumption is, that all user-specific Stuff resides in
 
    /srv/openhab/configurations       #the configs

    /srv/openhab/etc                  #for persistence-data

    /srv/openhab/webapps/images       #images for the UI

    /srv/openhab/webapps/static       #uuid & secret for my.openhab

    /srv/openhab/webapps/grafana      #grafana-stuff

    /srv/openhab/webapps/weather-data #weather-binding-stuff

which makes it necessary to move or copy the data first (only once).

All delivered `configurations/` are moved to `configurations_old/`, just as `etc/` is moved to `etc_old/` and `webapps/images/` is moved to `webapps/images_old/`, so no new data is lost (e.g. new openhab.cfg-entrys)

### Script 2

This script overwrites all files of the defined openhab-folder.
The packages you want to update can be specified in the script. (see filelist, addonlist).
You can also define file names to be excluded from the update process.

https://code.google.com/p/openhab-samples/source/browse/scripts/openhab_get_snapshot_overwrite.sh?repo=wiki

### Script 3

Add-on to old Version of Script 1 to install & configure the latest master of [HABmin](https://github.com/cdjackson/HABmin) as well as the rev of openHAB selected.

[openhab_habmin_get_snapshot.sh](https://code.google.com/p/openhab-samples/source/browse/scripts/openhab_habmin_get_snapshot.sh?repo=wiki)