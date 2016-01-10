## New & Noteworthy

Please find below the _intermediary_ release notes of the 1.8 Release.

### Version 1.8.0

See the Github issue tracker for a [full change log](https://github.com/openhab/openhab/issues?q=milestone%3A1.8.0).

####Major Features:
* [#3680](https://github.com/openhab/openhab/pull/3680) - [sallegra] Added Sallegra binding (replaces #3520) (@watou)
* [#3624](https://github.com/openhab/openhab/pull/3624) - initial checkin of the AKM868 on behalf of @MitchSUEW (@teichsta)
* [#3610](https://github.com/openhab/openhab/pull/3610) - InfluxDB persistence for influxdb versions >= 0.9  (@theoweiss)
* [#3569](https://github.com/openhab/openhab/pull/3569) - First Checkin of FritzBox TR064 Binding for 1.8.0 (@gitbock)
* [#3539](https://github.com/openhab/openhab/pull/3539) - Added Chamberlain MyQ Binding (@computergeek1507)
* [#3468](https://github.com/openhab/openhab/pull/3468) - caldav binding 26/11 (@querdenker2k)
* [#3401](https://github.com/openhab/openhab/pull/3401) - Sonance binding (@LaurensVanAcker)
* [#3386](https://github.com/openhab/openhab/pull/3386) - Implemented RWE Smarthome binding (@ollie-dev)
* [#3322](https://github.com/openhab/openhab/pull/3322) - InfluxDB persistence for influxdb versions >= 0.9 (@theoweiss)
* [#3266](https://github.com/openhab/openhab/pull/3266) - New DSC Alarm Action (@RSStephens)
* [#3248](https://github.com/openhab/openhab/pull/3248) - New generic JDBC Persistence Service (@lewie)
* [#3230](https://github.com/openhab/openhab/pull/3230) - Telegram action (@paolodenti)
* [#3229](https://github.com/openhab/openhab/pull/3229) - new binding for picnet home automation devices (@paolodenti)
* [#3187](https://github.com/openhab/openhab/pull/3187) - New binding: panStamp wireless Arduino modules (@GideonLeGrange)
* [#3086](https://github.com/openhab/openhab/pull/3086) - MQTT Action bundle (@kstaniek)
* [#3082](https://github.com/openhab/openhab/pull/3082) - Binding for relay boards available from http://www.ucprojects.eu (@rmichalak)
* [#3076](https://github.com/openhab/openhab/pull/3076) - Mystrom Binding (@coolweb)
* [#3002](https://github.com/openhab/openhab/pull/3002) - Added LCN binding (@Issendorff)
* [#2755](https://github.com/openhab/openhab/pull/2755) - Added Ecobee action bundle. (@watou)
* [#2707](https://github.com/openhab/openhab/pull/2707) - Initial contribution of SiteWhere persistence provider (@derekadams)
* [#2568](https://github.com/openhab/openhab/pull/2568) - Binding for octoller (www.octoller.com) V1.0.0 / Try 2 (@JPlenert)
* [#2567](https://github.com/openhab/openhab/pull/2567) - New binding to control RC switches (@mroeckl)
* [#2525](https://github.com/openhab/openhab/pull/2525) - Initial implementation of TACmi binding. (@twendt)
* [#2494](https://github.com/openhab/openhab/pull/2494) - FritzBoxTR064 Binding (@gitbock)
* [#2478](https://github.com/openhab/openhab/pull/2478) - New binding for IPX800 (@seebag)
* [#2293](https://github.com/openhab/openhab/pull/2293) - First contribution of AKM868 Binding (@MitchSUEW)
* [#2101](https://github.com/openhab/openhab/pull/2101) - new stiebel eltron heat pump binding (@kreutpet)

####Enhancements:
* [#3671](https://github.com/openhab/openhab/pull/3671) - Add support for energie measuerement on PWM-LAN devices. Replaces #2908 (@hmerk)
* [#3669](https://github.com/openhab/openhab/pull/3669) - Improve max cul support (@johgoe)
* [#3653](https://github.com/openhab/openhab/pull/3653) - Modbus binding: writing open/closed types now work with registers too (@ssalonen)
* [#3648](https://github.com/openhab/openhab/pull/3648) - Prevented Atmosphere from calling home - fixes #1527 (@digitaldan)
* [#3640](https://github.com/openhab/openhab/pull/3640) - Add type for maxCube binding to get informed about connection errors (@dominicdesu)
* [#3639](https://github.com/openhab/openhab/pull/3639) - Major Update eBus binding (docs) (@csowada)
* [#3636](https://github.com/openhab/openhab/pull/3636) - Major Update eBus binding (source) (@csowada)
* [#3621](https://github.com/openhab/openhab/pull/3621) - Insteon: Added Ramp Dimmer Support (@steve-bate)
* [#3615](https://github.com/openhab/openhab/pull/3615) - [novelanheatpump] Add port config parameter (for #3607) (@watou)
* [#3608](https://github.com/openhab/openhab/pull/3608) - Tinkerforge binding and action: new devices, enhancements and fixes (@theoweiss)
* [#3607](https://github.com/openhab/openhab/issues/3607) - Novelan / Luxtronic Port has changed (@boxerfahrer)
* [#3606](https://github.com/openhab/openhab/pull/3606) - InsteonPLM: Improved logging of items that match devices in modem database (@robnielsen)
* [#3604](https://github.com/openhab/openhab/pull/3604) - Plex binding update (@idserda)
* [#3598](https://github.com/openhab/openhab/pull/3598) - Weather: Added Wunderground total precipitation (@gerrieg)
* [#3593](https://github.com/openhab/openhab/pull/3593) - Weather: Fixed exception if degree is not in a valid range (@gerrieg)
* [#3434](https://github.com/openhab/openhab/pull/3434) - insteonplm: Added On/Off Outlet configuration (@SwissKid)
* [#3590](https://github.com/openhab/openhab/pull/3590) - InsteonPLM: removed comments after port configuration in openhab_default.cfg (@berndpfrommer)
* [#3588](https://github.com/openhab/openhab/pull/3588) - Updated Squeezebox speak action to allow configurable TTS URL (@sumnerboy12)
* [#3568](https://github.com/openhab/openhab/pull/3568) - [ecobee] Added quick poll after update, command or action. (@watou)
* [#3565](https://github.com/openhab/openhab/pull/3565) - a few fixes to run smoothly on Karaf (@kaikreuzer)
* [#3564](https://github.com/openhab/openhab/pull/3564) - Add Souliss types 54, 55, 56 and 58 (light, voltage, current, pressure). (@slepp)
* [#3553](https://github.com/openhab/openhab/pull/3553) - [Telegram action] enhancement: added send photo capability (@paolodenti)
* [#3552](https://github.com/openhab/openhab/pull/3552) - Tinkerforge binding and action: new devices, enhancements and fixes (@theoweiss)
* [#3544](https://github.com/openhab/openhab/pull/3544) - Wemo insight: additional parameters added #3538 (@toby200)
* [#3534](https://github.com/openhab/openhab/pull/3534) - OneWireBinding - Support for iButtons (@Dennis650)
* [#3515](https://github.com/openhab/openhab/pull/3515) - InsteonPLM: Include known features when an item is configured with an unknown feature (@robnielsen)
* [#3494](https://github.com/openhab/openhab/pull/3494) - [Ecobee] Updated to align with December 2015 ecobee API update. (@watou)
* [#3489](https://github.com/openhab/openhab/pull/3489) - [Sapp Binding] BigDecimal precision in scaling vs double (@paolodenti)
* [#3486](https://github.com/openhab/openhab/pull/3486) - Added support for regex substitutions to Regex transformation (@steve-bate)
* [#3485](https://github.com/openhab/openhab/pull/3485) - Log execution failures in ExecUtil. (@steve-bate)
* [#3476](https://github.com/openhab/openhab/pull/3476) - apt installation: openhab user as owner of /etc/openhab/configurations directory (@theoweiss)

####Bugfixes:
* [#3687](https://github.com/openhab/openhab/pull/3687) - Modbus binding: holding/input register state updates now support all the same item types as coil/discrete inputs (@ssalonen)
* [#3679](https://github.com/openhab/openhab/pull/3679) - remove the special toString() in order to sync with ESH (@teichsta)
* [#3677](https://github.com/openhab/openhab/pull/3677) - [caldav] fix logging and rescheduling (@querdenker2k)
* [#3676](https://github.com/openhab/openhab/pull/3676) - fix logging and rescheduling (@querdenker2k)
* [#3675](https://github.com/openhab/openhab/pull/3675) - Include binding fritzboxtr064 into build process (@gitbock)
* [#3670](https://github.com/openhab/openhab/pull/3670) - add bundles to maven build which will help increasing their version t… (@teichsta)
* [#3655](https://github.com/openhab/openhab/pull/3655) - Modbus binding: Items bound to discrete / coil issue fixed (@ssalonen)
* [#3646](https://github.com/openhab/openhab/pull/3646) - Update stale Bundle-Versions (for #3638) (@watou)
* [#3642](https://github.com/openhab/openhab/pull/3642) - Compare strings with equals method (@coolweb)
* [#3633](https://github.com/openhab/openhab/pull/3633) - [Hue] Osram-Par16-50-Workaround (@markusmazurczak)
* [#3605](https://github.com/openhab/openhab/pull/3605) - Better connection handling in pilight binding (@idserda)
* [#3566](https://github.com/openhab/openhab/pull/3566) - Added PersistenceService interface (@derekadams)
* [#3562](https://github.com/openhab/openhab/pull/3562) - Denon NPE fix (@idserda)
* [#3561](https://github.com/openhab/openhab/pull/3561) - Denon binding telnet connection fix (@idserda)
* [#3555](https://github.com/openhab/openhab/pull/3555) - Fixed session timeout problem (important for 1.8.0 release) (@ollie-dev)
* [#3552](https://github.com/openhab/openhab/pull/3552) - Tinkerforge binding and action: new devices, enhancements and fixes (@theoweiss)
* [#3551](https://github.com/openhab/openhab/pull/3551) - Fixes #3454: Use userdata for token path when running in OH2 (@hakan42)
* [#3547](https://github.com/openhab/openhab/pull/3547) - Specify pid file when checking process status (@jshprentz)
* [#3540](https://github.com/openhab/openhab/issues/3540) - Init.d reports incorrect openHAB status in Ubuntu 14.04 (@jshprentz)
* [#3532](https://github.com/openhab/openhab/pull/3532) - Updated start.bat to work on a path with spaces (@Tom-Davidson)
* [#3530](https://github.com/openhab/openhab/pull/3530) - jdbc persistence: fix postgresql table creation (@hakan42)
* [#3524](https://github.com/openhab/openhab/pull/3524) - [sapp binding] - binding not included in addons build (@paolodenti)
* [#3523](https://github.com/openhab/openhab/issues/3523) - Caldav binding: error executing event job (@TheNetStriker)
* [#3522](https://github.com/openhab/openhab/pull/3522) - Milight Binding - map a saturation value of 0 to WhiteMode, fixes #1400 (@hmerk)
* [#3516](https://github.com/openhab/openhab/issues/3516) - jdbc persistence: tableNamePrefix error on startup (@hakan42)
* [#3512](https://github.com/openhab/openhab/pull/3512) - Modbus binding to close connections on configuration refresh (@ssalonen)
* [#3507](https://github.com/openhab/openhab/pull/3507) - PointType needs hashCode and equals to be used in tests (@watou)
* [#3504](https://github.com/openhab/openhab/pull/3504) - [mqtt] Make inbound messages sensitive to item's accepted types and commands (@watou)
* [#3500](https://github.com/openhab/openhab/pull/3500) - mpd: fix artist and track not updated (@stefanroellin)
* [#3475](https://github.com/openhab/openhab/pull/3475) - fix permission and content directory issues when used within apt installation (@theoweiss)

####Removals:
* none

#### Major API changes
* none

## Updating the openHAB runtime 1.7 to 1.8

If you have a running openHAB runtime 1.7 installation, you can easily update it to version 1.8 by following these steps:
 1. Unzip the runtime 1.8 and all required addons to a new installation folder
 1. Replace the folder "configurations" by the version from your 1.7 installation
 1. Copy all other customizations you might have done to the new installation (e.g. additional images, sounds, etc.)

If you use the openHAB deb packages from the apt repository
 1. Add this line to your sources.list: "deb http://dl.bintray.com/openhab/apt-repo   stable    main“
 1. Open a command-line  and execute: apt-get update && apt-get upgrade

**NOTE: The upgrade process may overwrite your start script, so any custom modifications added to the openHAB launch command may need to be re-customized.**  An example of this is the use of named serial port instances with the option `-Dgnu.io.rxtx.SerialPorts=/dev/ttyAMA0`.  A restart may also be required.