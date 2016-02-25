Miscellaneous Tips & Tricks

* [How to redirect your log entries to the syslog](Samples-Tricks#how-to-redirect-your-log-entries-to-the-syslog)
* [How to do a proper ICMP ping on Linux](Samples-Tricks#how-to-do-a-proper-icmp-ping-on-linux)
* [How to add current or forecast weather icons to your sitemap](Samples-Tricks#how-to-add-current-or-forecast-weather-icons-to-your-sitemap)
* [How to configure openHAB to start automatically on Linux](Samples-Tricks#how-to-configure-openhab-to-start-automatically-on-linux)
* [How to configure openHAB as (Gentoo) openRC service](Samples-Tricks#how-to-configure-openhab-as-gentoo-openrc-service)
* [How to configure openHAB to start automatically on Linux with screen](Samples-Tricks#how-to-configure-openhab-to-start-automatically-on-linux-with-screen)
* [How to configure openHAB to start automatically on Windows](Samples-Tricks#how-to-configure-openhab-to-start-automatically-on-windows)
* [How start openHAB automatically on Linux using systemd](Samples-Tricks#how-start-openhab-automatically-on-linux-using-systemd)
* [How to configure openHAB to start automatically on MacOSX](Samples-Tricks#how-to-configure-openhab-to-start-automatically-on-macosx)
* [Use cheap bluetooth dongles on remote PCs to detect your phone/watch](Samples-Tricks#use-cheap-bluetooth-dongles-on-remote-pcs-to-detect-your-phonewatch)
* [Check presence by detecting WiFi phones/tablets] (Samples-Tricks#check-presence-by-detecting-wifi-phonestablets)
* [Get connection status of all network devices from Fritz!Box, eg for presence detection](Samples-Tricks#get-connection-status-of-all-network-devices-from-fritzbox-eg-for-presence-detection)
* [Use Fritzbox SIP Feature to make a call in case of an event](Samples-Tricks#use-fritzbox-sip-feature-to-make-a-call-in-case-of-an-event)
* [How to configure openHAB to connect to device symlinks (on Linux)](Samples-Tricks#how-to-configure-openhab-to-connect-to-device-symlinks-on-linux)
* [Use URL to manipulate items](Samples-Tricks#use-url-to-manipulate-items)
* [Extract caller and called number from Fritzbox Call object](Samples-Tricks#extract-caller-and-called-number-from-fritzbox-call-object)
* [Item loops with delay](Samples-Tricks#item-loops-with-delay)
* [enocean binding on Synology DS213+ (kernel driver package)](Samples-Tricks#enocean-binding-on-synology-ds213-kernel-driver-package)
* [How to use openHAB to activate or deactivate your Fritz!Box-WLAN](Samples-Tricks#how-to-use-openhab-to-activate-or-deactivate-your-fritzbox-wlan)
* [How to display Google Maps in a sitemap from a Mqttitude Mqtt message](Samples-Tricks#how-to-display-google-maps-in-a-sitemap-from-a-mqttitude-mqtt-message)
* [How to use Yahoo weather images](Samples-Tricks#how-to-use-yahoo-weather-images)
* [How to wake up with Philips Hue](Samples-Tricks#how-to-wake-up-with-philips-hue)
* [How to manage and sync configuration via subversion](Samples-Tricks#how-to-manage-and-sync-configuration-via-subversion)
* [How to switch LEDS on cubietruck](Samples-Tricks#how-to-switch-leds-on-cubietruck)
* [How to use a voice command from HABDroid](Samples-Tricks#how-to-use-a-voice-command-from-habdroid)
* [Add Humidex calculation for your Feels Like Temperature value](Samples-Tricks#add-humidex-calculation-for-your-feels-like-temperature-value)
* [Aeon Z-Stick Setup in Linux](Samples-Tricks#aeon-zstick-setup-in-linux) 
* [How to use a serial port under linux](Samples-Tricks#how-to-use-a-serial-port-under-linux)
* [Using the transceiver RFXtrx433E with Somfy RTS devices](Samples-Tricks#Using-the-transceiver-RFXtrx433E-with-Somfy-RTS-devices)
* [Talking to a Raspberry ZWAY device with push updates](Samples-Tricks#talking-to-a-raspberry-zway-device-with-push-updates)
* [How to use the LIFX beta API via executeCommandLine and curl](Samples-Tricks#how-to-use-the-lifx-beta-api-via-executecommandline-and-curl)
* [How to turn ON and OFF a Philips Hue with HomeMatic pushbutton](Samples-Tricks#how-to-turn-on-and-off-a-philips-hue-with-homematic-pushbutton)
* [How to create a simple Remote for controlling a Dune HD Player with Http and Denon Bindings](Samples-Tricks#how-to-create-a-simple-remote-for-controlling-a-dune-hd-player-with-http-and-denon-bindings)



### How to redirect your log entries to the syslog

You just need to add some lines to your logback.xml.

Like:
```xml
<appender name="SYSLOG" class="ch.qos.logback.classic.net.SyslogAppender">
   <syslogHost>localhost</syslogHost>
   <facility>AUTH</facility>
   <suffixPattern>{}openhab: [%thread] %logger %msg</suffixPattern>
</appender>
    
<root level="INFO">
   <appender-ref ref="SYSLOG" />
</root>
```
### How to do a proper ICMP ping on Linux

Java is not capable to open a raw socket to do a ICMP ping (see https://code.google.com/p/openhab/issues/detail?id=134). As a workaround, you can use the exec binding on Linux:

    Switch PingedItem { exec="<[/bin/sh@@-c@@ping -c 1 192.168.0.1 | grep \"packets transmitted\" | sed -e \"s/.*1 received.*/ON/\" -e \"s/.*0 received.*/OFF/\":30000:REGEX((.*))]" }

### How to add current or forecast weather icons to your sitemap

This example gets the weather information from the Wunderground online api service (for details see http://www.wunderground.com/weather/api/) and shows the current weather icon and weather state (such as partly cloudy) in one line like it would show any other Openhab item.
 
- in the items file, create a string as follows:
```
    String w   "Today [%s]"   <w>  { http="<[http://api.wunderground.com/api/<your api key>/forecast/q/Luxembourg/Luxembourg.xml:3600000:XSLT(wunderground_icon_forecast.xsl)]"} 
```
- copy the weather icons to your images folder and rename and convert them to png, e.g. "w-partly_cloudy.png". The weather icon sets can be found here:

    http://www.wunderground.com/weather/api/d/docs?d=resources/icon-sets 

- create a stylesheet named wunderground_icon_forecast.xsl containing the following lines (place it under configurations/transform):
```
<?xml version="1.0"?>
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform" version="2.0">       
   <xsl:output indent="yes" method="xml" encoding="UTF-8" omit-xml-declaration="yes" />
   <xsl:template match="/">
      <!-- format: hh:mm:ss -->
      <xsl:value-of select="//response/forecast/txt_forecast/forecastdays/forecastday/icon/text()" />
   </xsl:template>
</xsl:stylesheet>
```
- display the string in the .sitemap file:

`Text item=w `

### How to configure openHAB to start automatically on Linux

Create a new file in /etc/init.d/openhab using your preferred editor (e.g. nano) and copy the code below (make sure to replace RUN_AS with the user you are running OpenHab as). 

```sh
#! /bin/sh
### BEGIN INIT INFO
# Provides:          openhab
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: OpenHAB Daemon
### END INIT INFO

# Author: Thomas Brettinger 

# Do NOT "set -e"

# PATH should only include /usr/* if it runs after the mountnfs.sh script
PATH=/sbin:/usr/sbin:/bin:/usr/bin

DESC="Open Home Automation Bus Daemon"
NAME=openhab
DAEMON=/usr/bin/java
PIDFILE=/var/run/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
ECLIPSEHOME="/opt/openhab";
HTTPPORT=8080
HTTPSPORT=8443
TELNETPORT=5555
OPENHAB_LOG=/var/log/openhab.log
# be sure you are adopting the user to your local OH user 
RUN_AS=ben

# make sure logfile has proper owner
touch $OPENHAB_LOG && chown $RUN_AS $OPENHAB_LOG

# get path to equinox jar inside $eclipsehome folder
cp=$(find $ECLIPSEHOME/server -name "org.eclipse.equinox.launcher_*.jar" | sort | tail -1);

DAEMON_ARGS="-Dosgi.clean=true -Declipse.ignoreApp=true -Dosgi.noShutdown=true -Djetty.port=$HTTPPORT -Djetty.port.ssl=$HTTPSPORT -Djetty.home=$ECLIPSEHOME -Dlogback.configurationFile=$ECLIPSEHOME/configurations/logback.xml -Dfelix.fileinstall.dir=$ECLIPSEHOME/addons -Djava.library.path=$ECLIPSEHOME/lib -Djava.security.auth.login.config=$ECLIPSEHOME/etc/login.conf -Dorg.quartz.properties=$ECLIPSEHOME/etc/quartz.properties -Djava.awt.headless=true -jar $cp -console ${TELNETPORT}"

# Exit if the package is not installed
[ -x "$DAEMON" ] || exit 0

# Read configuration variable file if it is present
[ -r /etc/default/$NAME ] && . /etc/default/$NAME

# Load the VERBOSE setting and other rcS variables
. /lib/init/vars.sh

# Define LSB log_* functions.
# Depend on lsb-base (>= 3.2-14) to ensure that this file is present
# and status_of_proc is working.
. /lib/lsb/init-functions

#
# Function that starts the daemon/service
#
do_start()
{
    # Return
    #   0 if daemon has been started
    #   1 if daemon was already running
    #   2 if daemon could not be started
    start-stop-daemon --start --quiet --make-pidfile --pidfile $PIDFILE --chuid $RUN_AS --chdir $ECLIPSEHOME --exec $DAEMON --test > /dev/null \
        || return 1
    start-stop-daemon --start --quiet --background --make-pidfile --pidfile $PIDFILE --chuid $RUN_AS --chdir $ECLIPSEHOME  --startas /bin/bash -- -c "exec $DAEMON $DAEMON_ARGS > $OPENHAB_LOG 2>&1" \
        || return 2
    # Add code here, if necessary, that waits for the process to be ready
    # to handle requests from services started subsequently which depend
    # on this one.  As a last resort, sleep for some time.
    return 0
}

#
# Function that stops the daemon/service
#
do_stop()
{
    # Return
    #   0 if daemon has been stopped
    #   1 if daemon was already stopped
    #   2 if daemon could not be stopped
    #   other if a failure occurred
    start-stop-daemon --stop --quiet --retry=TERM/30/KILL/5 --pidfile $PIDFILE --name $NAME
    RETVAL="$?"
    [ "$RETVAL" = 2 ] && return 2
    # Wait for children to finish too if this is a daemon that forks
    # and if the daemon is only ever run from this initscript.
    # If the above conditions are not satisfied then add some other code
    # that waits for the process to drop all resources that could be
    # needed by services started subsequently.  A last resort is to
    # sleep for some time.
    start-stop-daemon --stop --quiet --oknodo --retry=0/30/KILL/5 --exec $DAEMON
    [ "$?" = 2 ] && return 2
    # Many daemons don't delete their pidfiles when they exit.
    rm -f $PIDFILE
    return "$RETVAL"
}

#
# Function that sends a SIGHUP to the daemon/service
#
do_reload() {
    #
    # If the daemon can reload its configuration without
    # restarting (for example, when it is sent a SIGHUP),
    # then implement that here.
    #
    do_stop
    sleep 1
    do_start
    return 0
}

case "$1" in
  'start')
    log_daemon_msg "Starting $DESC"
    do_start
    case "$?" in
        0|1) log_end_msg 0 ;;
        2) log_end_msg 1 ;;
    esac
    ;;
  'stop')
    log_daemon_msg "Stopping $DESC" 
    do_stop
    case "$?" in
        0|1) log_end_msg 0 ;;
        2) log_end_msg 1 ;;
    esac
    ;;
  'status')
       status_of_proc "$DAEMON" "$NAME" && exit 0 || exit $?
       ;;
  #reload|force-reload)
    #
    # If do_reload() is not implemented then leave this commented out
    # and leave 'force-reload' as an alias for 'restart'.
    #
    #log_daemon_msg "Reloading $DESC" "$NAME"
    #do_reload
    #log_end_msg $?
    #;;
  'restart'|'force-reload')
    #
    # If the "reload" option is implemented then remove the
    # 'force-reload' alias
    #
    log_daemon_msg "Restarting $DESC" 
    do_stop
    case "$?" in
      0|1)
        do_start
        case "$?" in
            0) log_end_msg 0 ;;
            1) log_end_msg 1 ;; # Old process is still running
            *) log_end_msg 1 ;; # Failed to start
        esac
        ;;
      *)
          # Failed to stop
        log_end_msg 1
        ;;
    esac
    ;;
  *)
    #echo "Usage: $SCRIPTNAME {start|stop|restart|reload|force-reload}" >&2
    echo "Usage: $SCRIPTNAME {start|stop|status|restart|force-reload}" >&2
    exit 3
    ;;
esac
:
```
Make the script executable and configure it to run on boot.

    sudo chmod a+x /etc/init.d/openhab
    sudo update-rc.d openhab defaults

Now whenever your Linux machine boots openHAB will be automatically started.

Note: You may find that the script will not run on startup because it requires a parameter passed at the command line. Use the following to edit the rc.local file.
```
sudo nano /etc/rc.local
```
Then add these lines anywhere above the exit 0
```
# start openhab
/etc/init.d/openhab start
```
Save it and exit.  That should do it.

### How to configure openHAB as (Gentoo) openRC service

Create the openhab user with

```sh
useradd -s /sbin/nologin openhab
```

and the log directory

```sh
mkdir /var/log/openhab && chown openhab /var/log/openhab
```

then create `/etc/(conf|init).d/openhab` as below and add it to the default runlevel

```sh
rc-update add openhab default
```

OpenHAB must be installed in `/opt/openhab`

`/etc/init.d/openhab`:

```sh
#!/sbin/runscript

PIDFILE=/var/run/openhab.pid

OPENHAB_HOME=/opt/openhab
OPENHAB_LOGDIR=/var/log/openhab
OPENHAB_LOG=$OPENHAB_LOGDIR/openhab.log

EQUINOX_JAR=$(find $OPENHAB_HOME/server -name "org.eclipse.equinox.launcher_*.jar" | sort | tail -1)

OPENHAB_ARGS="
-Dosgi.clean=true \
-Declipse.ignoreApp=true \
-Dosgi.noShutdown=true \
-Djetty.port=$HTTP_PORT
-Djetty.port.ssl=$HTTPS_PORT \
-Djetty.home=. \
-Dlogback.configurationFile=configurations/logback.xml \
-Dfelix.fileinstall.dir=addons \
-Djava.library.path=lib \
-Djava.security.auth.login.config=./etc/login.conf \
-Dorg.quartz.properties=./etc/quartz.properties \
-Dequinox.ds.block_timeout=240000 \
-Dequinox.scr.waitTimeOnBlock=60000 \
-Dfelix.fileinstall.active.level=4 \
-Djava.awt.headless=true \
-jar $EQUINOX_JAR"

depend() {
	need net
}

checkconfig() {
	if [ ! -d "$OPENHAB_LOGDIR" ]; then
		mkdir "$OPENHAB_LOGDIR"
		chown $USER "$OPENHAB_LOGDIR"
	fi
}

start() {
	checkconfig
	ebegin "Starting openHAB"
	start-stop-daemon \
		--start \
		--user $USER \
		--group $GROUP \
		--pidfile $PIDFILE \
		--make-pidfile \
		--chdir "$OPENHAB_HOME" \
		--stdout $OPENHAB_LOG \
		--stderr $OPENHAB_LOG \
		--background \
		--exec java -- $OPENHAB_ARGS
	eend $?
}

stop() {
	ebegin "Stopping openHAB"
	start-stop-daemon \
		--stop \
		--pidfile $PIDFILE \
		--retry "SIGTERM/15 SIGKILL/30" \
		--progress
	eend $?
}

status() {
	[ ! -f "$PIDFILE" ] && return 1

	if ps -p $(cat "$PIDFILE") > /dev/null; then
		eend 0
	else
		eend 1
	fi
}
```

`/etc/conf.d/openhab`:

```sh
USER=openhab

GROUP=nogroup

HTTP_PORT=8081

HTTPS_PORT=8443
```

### How to configure openHAB to start automatically on Linux with screen

Prequerities: screen installed

```sh
    #! /bin/sh
### BEGIN INIT INFO
# Provides:   openHAB
# Required-Start: $local_fs $remote_fs
# Required-Stop:  $local_fs $remote_fs
# Should-Start:   $network
# Should-Stop:    $network
# Default-Start:  2 3 4 5
# Default-Stop:   0 1 6
# Short-Description:    Start and stop openHAB in screen Session
# Description:    This runs openHAB continuously in screen.
### END INIT INFO
# Set OH-User
OHUSER=openhab
# Set OH-Path
OHPATH=/opt/openhab

# uncomment and set if necessary to obtain correct Codepage and language
# export LC_ALL=de_DE.UTF-8

case "$1" in

  start)
        PID=`ps -ef | grep openHAB | grep -v grep | awk '{print $2}'`
        if [ "${PID}" != "" ]
         then
          echo openHAB-Screen scheint schon zu laufen! PID ist `pidof SCREEN`
         else
          echo "Starting openHAB"
          cd ${OHPATH}
          sudo -u ${OHUSER} screen -S openHAB -dm  sh ./start.sh
        fi
        ;;
  stop)
        echo "Stopping openHAB"
        sudo -u ${OHUSER} screen -S openHAB -p 0 -X stuff "exit$(printf \\r)"
        sudo -u ${OHUSER} screen -S openHAB -p 0 -X stuff "y$(printf \\r)"
        sudo -u ${OHUSER} screen -S openHAB -p 0 -X stuff "exit$(printf \\r)"
        PID=`ps -ef | grep openHAB | grep -v grep | awk '{print $2}'`
        while [ `ps -ef | grep $PID | wc -l` -gt 1 ]
         do
          echo -n .
          sleep 2
         done
        echo .
        ;;
  restart|force-reload)
        echo "Restarting openHAB"
        $0 stop
        $0 start
        ;;
  *)
        N=/etc/init.d/$NAME
        echo "Usage: $N {start|stop|restart}" >&2
        exit 1
        ;;
esac
exit 0
```
Make the script executable and configure it to run on boot.

    sudo chmod a+x /etc/init.d/openhab
    sudo update-rc.d openhab defaults

Now whenever your Linux machine boots openHAB will be automatically started in a screen-session called openHAB. To reach the OSGI-Console, just type 

    sudo -u openhab screen -r openHAB

### How to configure openHAB to start automatically on Windows

This setup uses procmon from Apache. Download the binaries from http://www.apache.org/dist/commons/daemon/binaries/windows/. Unzip to the runtime folder
Create a install_service.bat file, it will be based on start.bat, see below.
I must be adapted to match your version/path to openhab.
First time you will have to go into windows services and reenter the login info for the service, this will grant the logon as service right to the user.
```
set ECLIPSEHOME=server

:: set ports for HTTP(S) server
set HTTP_PORT=8080
set HTTPS_PORT=8443
set FOLDER=C:\ServerFolders\openhab\runtime
set JAVA_HOME=C:\Program Files (x86)\Java\jdk1.7.0_51
prunsrv //DS//OpenHab
prunsrv //IS//OpenHab --DisplayName="OpenHab Service" ^
	--ServiceUser jbh@NB1.local --ServicePassword XXXXXXXXX^
	--Classpath "%FOLDER%\server\plugins\org.eclipse.equinox.launcher_1.3.0.v20120522-1813.jar;%FOLDER%\server\plugins\org.eclipse.scada.utils.osgi.daemon_0.1.0.v20140430-1536.jar"^
        --Install=%FOLDER%\prunsrv.exe --Jvm=auto ^
       --StartMode jvm^
		--StartClass org.eclipse.equinox.launcher.Main^
  --StartMethod main^
  --Startup auto^
  ++StartParams -console^
  ++StartParams -consoleLog^
   ++JvmOptions -Dosgi.clean=true#-Declipse.ignoreApp=true#-Dosgi.noShutdown=true#-Djetty.port=%HTTP_PORT%#-Djetty.port.ssl=%HTTPS_PORT%#-Djetty.home=.#-Dlogback.configurationFile=configurations/logback.xml#-Dfelix.fileinstall.dir=addons#-Djava.library.path=lib#-Djava.security.auth.login.config=./etc/login.conf#-Dorg.quartz.properties=./etc/quartz.properties#-Dequinox.ds.block_timeout=240000#-Dequinox.scr.waitTimeOnBlock=60000#-Djava.awt.headless=true#-Dfelix.fileinstall.active.level=4^
  --StopMode jvm^
  --LogPath %FOLDER%\logs
pause
```

### How start openHAB automatically on Linux using systemd

Systemd is used as init system on various Linux flavors, i.e. Debian, Ubuntu, openSUSE, ArchLinux, etc.
To make openHAB start automatically on system startup we utilize the main start.sh script which comes with the openHAB distribution.
We just need to modify it a bit.
Instead of this line:
```
java -Dosgi.clean=true -Declipse.ignoreApp=true -Dosgi.noShutdown=true -Djetty.port=$HTTP_PORT -Djetty.port.ssl=$HTTPS_PORT -Djetty.home=. -Dlogback.configurationFile=configurations/logback.xml -Dfelix.fileinstall.dir=addons -Djava.library.path=lib -Djava.security.auth.login.config=./etc/login.conf -Dorg.quartz.properties=./etc/quartz.properties -Dequinox.ds.block_timeout=240000 -Dequinox.scr.waitTimeOnBlock=60000 -Dfelix.fileinstall.active.level=4 -Djava.awt.headless=true -jar $cp $* -console 
```
we'll use:
```
java -Dosgi.clean=true -Declipse.ignoreApp=true -Dosgi.noShutdown=true -Djetty.port=$HTTP_PORT -Djetty.port.ssl=$HTTPS_PORT -Djetty.home=. -Dlogback.configurationFile=configurations/logback.xml -Dfelix.fileinstall.dir=addons -Djava.library.path=lib -Djava.security.auth.login.config=./etc/login.conf -Dorg.quartz.properties=./etc/quartz.properties -Dequinox.ds.block_timeout=240000 -Dequinox.scr.waitTimeOnBlock=60000 -Dfelix.fileinstall.active.level=4 -Djava.awt.headless=true -jar $cp > run.log 2>&1 &
```
this makes openHAB run in the background and redirect stderr and stdout to a file called run.log.
Notice that only the end of the line has changed.
You'll also may want to create a copy of the original script file before making the change.

Now here is what we need for systemd:
Identify where the service units are placed on your system. In openSUSE it is: ```/usr/lib/systemd```.
Now create a new file called ```openhab.service``` in this folder: ```/usr/lib/systemd/system```
and copy the following into it:
```
[Unit]
Description=Open Home Automation Bus

[Service]
Type=forking
GuessMainPID=yes
ExecStart=/Space/Apps/openhab/start.sh
ExecStop=/usr/bin/kill -SIGINT $MAINPID
ExecStopPost=
User=openhab
Group=daemon
WorkingDirectory=/Space/Apps/openhab

[Install]
WantedBy=multi-user.target
```
Adjust the WorkingDirectory and the path to start.sh as you need.
Also this unit uses a separate user to run openHAB with. This user must be created first.
You can as well start openHAB with your own user account.
The Type ```forking``` must be used here because of the change we did in the start.sh script.

If that is done "install" the unit:
```systemctl enable openhab.service```

Now start the service:
```systemctl start openhab.service```

You may check if it's running with:
```systemctl status openhab.service```

You should see something this if it's running:
```
openhab.service - Open Home Automation Bus
   Loaded: loaded (/usr/lib/systemd/system/openhab.service; enabled)
   Active: active (running) since Do 2014-05-01 11:54:41 CEST; 1h 23min ago
  Process: 6118 ExecStart=/Space/Apps/openhab/start.sh (code=exited, status=0/SUCCESS)
 Main PID: 6124 (java)
   CGroup: /system.slice/openhab.service
           └─6124 java -Dosgi.clean=true -Declipse.ignoreApp=true -Dosgi.noShutdown=true -Djetty.port=8080 -Djetty.port.ssl=8443 -Djetty.home=. -Dlogback.configurationFi...
```

### How to configure openHAB to start automatically on MacOSX

Create a new file named org.openhab.daemon.plist in /Library/LaunchDaemons using your preferred editor (e.g. nano) as root user.

```
$ sudo nano /Library/LaunchDaemons/org.openhab.daemon.plist
```

Copy the code below. 

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>GID</key>
        <integer>0</integer>
        <key>Label</key>
        <string>org.openhab.daemon</string>
        <key>OnDemand</key>
        <false/>
        <key>RunAtLoad</key>
        <true/>
        <key>Program</key>
        <string>/Applications/openhab/start.sh</string>
        <key>UserName</key>
        <string>root</string>
        <key>KeepAlive</key>
        <true/>
</dict>
</plist>
```

Ensure that the file has the correct user.

```
$ sudo chown root /Library/LaunchDaemons/org.openhab.daemon.plist
```

The openhab process will now be launched at startup by launchd.

Alternatively it is possible to start the openhab process with the following command:

```
$ sudo launchctl load -w /Library/LaunchDaemons/org.openhab.daemon.plist
```

It is also possible to stop the openhab process using the following command:

```
$ sudo launchctl unload /Library/LaunchDaemons/org.openhab.daemon.plist
```

### Use cheap bluetooth dongles on remote PCs to detect your phone/watch

This is more an idea really but consider that openhab can know your home by picking up on the bluetooth on your phone, using cheap 3 euro bluetooth dongles connected to the various PCs and media centres you may have.

EG On an ubuntu/debian based PC.

1) Install the "bluez" bluetooth stack.

```sh
apt-get install bluez
```
2) Use the bash script below to do a simple scan several times per min.  This script looks for my phone "jon-phone".  Its made no real difference to the battery life on my android 4.x phone.  The script updates a switch in openhab called "zoneOneState".  You could have as many zones as you need/want to cover your house.
```sh
    #!/bin/bash
    
    while true
    do
      DEVICES=`hcitool scan`
    
      if [[ $DEVICES = *jon-phone* ]]
      then
        curl --max-time 2 --connect-timeout 2 --header "Content-Type: text/plain" --request PUT --data "ON" http://192.168.1.121:8080/rest/items/zoneOneState/state
      else
        curl --max-time 2 --connect-timeout 2 --header "Content-Type: text/plain" --request PUT --data "OFF" http://192.168.1.121:8080/rest/items/zoneOneState/state
      fi
    
      sleep 30
    done
```
3) The rule governing the decision process on if your house is occupied can be as complex or simple as you need.  The rule below is clunky but simple and gets the job done.  It uses three zone switches to decide if it should set a switch called "occupiedState" to on or off.  Then using the occupiedState switch you can make your other openhab rules behave in different ways.  Each zone checked reports true if your bluetooth device has been seen within the last 5 mins (helps stop flap caused by the patchy coverage of bluetooth signals).

    //FIGURE IF HOUSE OCCUPIED EVERY 30 SECONDS
    rule "check occupiedState"
    when
    	Time cron "*/30 * * * * ?"
    then
    	if (occupiedState.state == ON)
    	{
    		zoneOneFlag = true
    		zoneTwoFlag = true
    		zoneThreeFlag = true
    
    		if (zoneOneState.state == OFF)
    		{
    			if (!zoneOneState.changedSince (now.minusMinutes(5)))
    			{
    				//sendCommand("occupiedState", "OFF")
    				zoneOneFlag = false
    			}
    		}
    		if (zoneTwoState.state == OFF)
    		{
    			if (!zoneTwoState.changedSince (now.minusMinutes(5)))
    			{
    				//sendCommand("occupiedState", "OFF")
    				zoneTwoFlag = false
    			}
    		}
    		if (zoneThreeState.state == OFF)
    		{
    			if (!zoneThreeState.changedSince (now.minusMinutes(5)))
    			{
    				//sendCommand("occupiedState", "OFF")
    				zoneThreeFlag = false
    			}
    		}
    
    		if (zoneOneFlag == false && zoneTwoFlag == false && zoneThreeFlag == false) {
    			sendCommand("occupiedState", "OFF")
    		}
    	} else {
    		if (zoneOneState.state == ON || zoneTwoState.state == ON || zoneThreeState.state == ON) {
    			sendCommand("occupiedState", "ON")
    		}
    	}

### Check presence by detecting WiFi phones/tablets
This is a small modification to the tip above. This time we check, whether somebody is at home by testing if their phone is within WiFi reach.
All you need is the Network Health Binding.

Your items file could look like this:

    Group gMobiles
    Switch Presence
    Switch phone1 (gMobiles) {nh="android-xxxxx"}
    Switch phone2 (gMobiles) {nh="android-yyyyy"}

My rules file looks like this:

    rule "Periodically check presence"
    when
        Time cron "0 */5 * * * ?"
    then
            if (Presence.state == ON)
            {
                    if(gMobiles.members.filter(s | s.state == ON).size == 0) {
                            logInfo("PresenceCheck", "No phone within reach, checking for flapping")
                            if(gMobiles.members.filter(s | s.changedSince(now.minusMinutes(5))).size == 0) {
                                    logInfo("PresenceCheck", "Nobody is at home")
                                    sendCommand(Presence, OFF)
                            }
                    }
            }
            else
            {
                    //For initialisation. If Presence is undefined or off, although it should be on.
                    if(gMobiles.members.filter(s | s.state == ON).size > 0) {
                            sendCommand(Presence, ON)
                    }
                    else if (Presence.state == Undefined || Presence.state == Uninitialized) {
                            sendCommand(Presence, OFF)
                    }
            }
    
    end
    
    rule "Coming home"
    when
            Item gMobiles changed
    then
            if (Presence.state != ON) {
                    if(gMobiles.members.filter(s | s.state == ON).size > 0) {
                            logInfo("PresenceCheck", "Somebody is home")
                            sendCommand(Presence, ON)
                    }
            }
    end

Presence is only switched off, if no phone was seen within the last 5 minutes. This avoids flapping, if you
are just taking out the trash or something.

Additionally to the Network Health items in the gMobiles group, you could add bluetooth devices (like used above) or motion sensors.

### Get connection status of all network devices from Fritz!Box, eg for presence detection

For cases where bluetooth detection described above or network health binding is not suitable.

**Requirements**

- Fritz!Box with telnet access
- Linux server (preferably the openHAB server), tested with ReadyNAS
- Linux knowledge to adopt scripts

**Methodology**

- Shell script on the Fritz!Box to dump all known network devices with their current connection status
- This script is called from the Linux server through a cron job periodically (eg every minute), parses the dump and updates openHAB items according to their connection status

**Restrictions**

- Only tested on ReadyNAS Linux with manual added packages, others might need adoption
- All parameters like login/password/paths/script names "hard-coded" into the scripts, need adoption
- Absolutely **no** error-checks, use at own risk
- Due to these restrictions only applicable for people with at least basic Linux knowledge

**How-To**

#### On the Fritz!Box

The Fritz!Box shows a reliable status of devices connected to the network using the built-in _ctlmgr_ctl r landevice_ command (see http://www.wehavemorefun.de/fritzbox/Landevice). 

Create the script *landevices.sh* permanently (should not be deleted at reboot), for example in folder */var/media/ftp/Fritz!Box/*. Make it executable. Take care not to indent the script as the Fritz!Box does not like that.
```sh
    #!/bin/sh
    i=0
    count_devices=`ctlmgr_ctl r landevice settings/landevice/count`
    while [ $i -lt $count_devices ]
    do
    device="landevice="$i
    #for command in name ip mac manu_name dhcp static_dhcp wlan ethernet active online speed deleteable wakeup source neighbour_name is_double_neighbour_name ipv6addrs ipv6_ifid
    for command in name active
    do
    output="`ctlmgr_ctl r landevice settings/landevice$i/$command`"
    device=$device" "$command"="$output
    done
    echo $device
    i=`expr $i + 1`
    done
```
The commented out for loop shows all possible subcommands, so it is also possible to output eg the speed or mac adress of all landevices. However this would also require a change on the calling script below. Once created on the Fritz!Box and executed it should provide you all known network devices with their current status:
```sh
    # ./landevices.sh
    landevice=0 name=BoyLaptop active=0
    landevice=1 name=BoyPhone active=0
    landevice=2 name=BoyiPod active=0
    landevice=3 name=DGOfficePrinter active=1
    landevice=4 name=DGTVRasPlex active=1
    landevice=5 name=DGTVWii active=0
    landevice=6 name=DadLaptop active=0
    landevice=7 name=DadLaptopWork active=0
    landevice=8 name=DadLaptopWork active=0
    landevice=9 name=DadPC active=1
    landevice=10 name=DadPhone active=1
    landevice=11 name=DadPhoneWork active=1
    landevice=12 name=EGEntranceAP active=1
    landevice=13 name=EGTVRasPlex active=1
    landevice=14 name=EGWZAP active=0
    landevice=15 name=GirlLaptop active=0
    landevice=16 name=GirlPhone active=1
    landevice=17 name=GuestLaptop active=0
    landevice=18 name=KGOfficePrinter active=1
    landevice=19 name=KGOfficeReadyNAS active=1
    landevice=20 name=KGTEC1AP active=1
    landevice=21 name=KGTEC2AP active=1
    landevice=22 name=MomLaptop active=0
    landevice=23 name=MomLaptop active=0
    landevice=24 name=MomLaptopWork active=0
    landevice=25 name=MomPhone active=0
```
If required, use the Fritz!Box GUI to give the devices some reasonable names in order to easily assign phones/laptops etc to people or media devices to floors/rooms. Using the _ctlmgr_ctl_ command it would also be possible to only query for specific devices (go through the full list and look for either name or mac or both), but any change in the output of this script requires changes on the other end as well.

#### On the Linux server

Create a *expect* script in a folder on your Linux Server and make it executable. The only purpose of this script is only to connect to your Fritz!Box through a telnet session and call the script created there.
```sh
    #!/usr/bin/expect
    spawn telnet <<<your fritzbox ip here>>>
    expect "user: "
    send "<<<your fritzbox login here>>>\r"
    expect "password: "
    send "<<<your fritzbox password here>>>\r"
    expect "telnet"
    expect "#"
    send "\r"
    expect "#"
    send "/var/media/ftp/Fritz!Box/landevices.sh\r"
    expect "#"
    send "exit 0\r"
```
Here the IP of the Fritz!Box, your credentials and the path to the script need to be adopted. Furthermore the above script is for the newest version of the Fritz!box firmware where multiple user accounts can be created. In case this is not your case, you have to omit the "user:" and "login" *expect/send* statements in the script above, as in that case the Fritz!Box telnet session only requires the password, not the user. Again, to validate the functionality just execute the script:
```sh
    ReadyNAS:~# ./fritzbox_devices.expect
    spawn telnet 192.168.178.1
    
    Entering character mode
    Escape character is '^]'.
    
    Fritz!Box user: admin
    password:
    
    
    BusyBox v1.20.2 (2013-05-13 12:53:07 CEST) built-in shell (ash)
    Enter 'help' for a list of built-in commands.
    
    ermittle die aktuelle TTY
    tty is "/dev/pts/1"
    weitere telnet Verbindung aufgebaut
    #
    # /var/media/ftp/Fritz!Box/landevices.sh
    landevice=0 name=BoyLaptop active=0
    landevice=1 name=BoyPhone active=0
    landevice=2 name=BoyiPod active=0
    landevice=3 name=DGOfficePrinter active=1
    landevice=4 name=DGTVRasPlex active=1
    landevice=5 name=DGTVWii active=0
    landevice=6 name=DadLaptop active=0
    landevice=7 name=DadLaptopWork active=0
    landevice=8 name=DadLaptopWork active=0
    landevice=9 name=DadPC active=1
    landevice=10 name=DadPhone active=1
    landevice=11 name=DadPhoneWork active=1
    landevice=12 name=EGEntranceAP active=1
    landevice=13 name=EGTVRasPlex active=1
    landevice=14 name=EGWZAP active=0
    landevice=15 name=GirlLaptop active=0
    landevice=16 name=GirlPhone active=1
    landevice=17 name=GuestLaptop active=0
    landevice=18 name=KGOfficePrinter active=1
    landevice=19 name=KGOfficeReadyNAS active=1
    landevice=20 name=KGTEC1AP active=1
    landevice=21 name=KGTEC2AP active=1
    landevice=22 name=MomLaptop active=0
    landevice=23 name=MomLaptop active=0
    landevice=24 name=MomLaptopWork active=0
    landevice=25 name=MomPhone active=0
    # ReadyNAS:~#
```
As you can see here we have the original output of the script plus some "overhead" for the telnet session.

Now create a *sed* script _fritzbox_devices.sed_ that will create *curl* commands to interface with the openHAB server:
```sh
    #!/bin/sed -f
    s#\r$##
    s#landevice=[0-9]* name=#curl --silent -H \"Content-Type: text/plain\" http://localhost:8080/rest/items/landevice_#g
    s#active=0#-d \"OFF\"#g
    s#active=1#-d \"ON\"#g
```
The first line here is just to eliminate "DOS-like" \r at the end of each line which show up with a ReadyNAS as server and might not be required in your setup, but you can still leave them. The second line replaces the "landevice=....name=...... with a *curl* command. In order to have reasonable variable names again in openHAB the landevices there start with "landevice_", so "DadPhone" on the Fritz!Box gets landevice_DadPhone in openHAB. Furthermore *localhost:8080* needs to be changed to match your openHAB server if is not running on the same machine. The last two lines replace the "active=0" or "active=1" with the correct statements for *curl*.

Another validation of the scripts created so far:

    ReadyNAS:~# ./fritzbox_devices.expect | fgrep "landevice=" | ./fritzbox_devices.sed
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_BoyLaptop -d "OFF"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_BoyPhone -d "ON"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_BoyiPod -d "OFF"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DGOfficePrinter -d "ON"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DGTVRasPlex -d "ON"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DGTVWii -d "OFF"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DadLaptop -d "OFF"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DadLaptopWork -d "OFF"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DadLaptopWork -d "OFF"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DadPC -d "ON"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DadPhone -d "ON"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DadPhoneWork -d "ON"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_EGEntranceAP -d "ON"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_EGTVRasPlex -d "ON"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_EGWZAP -d "OFF"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_GirlLaptop -d "OFF"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_GirlPhone -d "ON"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_GuestLaptop -d "OFF"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_KGOfficePrinter -d "ON"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_KGOfficeReadyNAS -d "ON"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_KGTEC1AP -d "ON"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_KGTEC2AP -d "ON"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_MomLaptop -d "OFF"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_MomLaptop -d "OFF"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_MomLaptopWork -d "OFF"
    curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_MomPhone -d "OFF"
    ReadyNAS:~#

#### Configuration in openHAB

Now it is time to create the required items in openHAB - One Switch for each landevice that is supposed to be used later. In case you don't configure all landevices as items you will get error messages in openHAB logs, however these can be ignored. Just to have a "clean" configuration you might want to add all devices.

    Switch landevice_KGOfficeReadyNAS  "ReadyNAS"                       <network> (KG_TEC1, gNetworkServer)
    Switch landevice_EGEntranceAP      "Airport Erdgeschoss"            <network> (gWZ, gNetworkServer)
    Switch landevice_EGWZAP            "Airport Repeater Erdgeschoss"   <network> (gWZ, gNetworkServer)
    Switch landevice_KGTEC1AP          "Airport Keller TEC1"            <network> (KG_TEC1, gNetworkServer)		
    Switch landevice_KGTEC2AP          "Airport Keller TEC2"            <network> (KG_TEC2, gNetworkServer)
    Switch landevice_DGOfficePrinter   "Drucker Dachgeschoss"           <network> (DG_GUEST, gNetworkMedia)
    Switch landevice_KGOfficePrinter   "Drucker Keller"                 <network> (KG_TEC1, gNetworkMedia)
    Switch landevice_DGTVRasPlex		"RasPLEX Dachgeschoss"	<network> (DG_TV, gNetworkMedia)
    Switch landevice_EGTVRasPlex		"RasPLEX Erdgeschoss"	<network> (EG_TV, gNetworkMedia)
    Switch landevice_DGTVWii		"Wii"			<network> (DG_TV, gNetworkMedia)
    Switch landevice_DadPC			"Dad PC"		<network> (gWZ, gNetworkDad)
    Switch landevice_DadLaptop		"Dad Laptop"		<network> (gWZ, gNetworkDad)
    Switch landevice_DadPhone		"Dad Phone"		<network> (gWZ, gDAD_PRESENT, gNetworkDad, gPhones)
    Switch landevice_DadLaptopWork	        "Dad Laptop@work"	<network> (gWZ, gNetworkDad)
    Switch landevice_DadPhoneWork	        "Dad Phone@work"	<network> (gWZ, gDAD_PRESENT, gNetworkDad, gPhones)
    Switch landevice_MomLaptop		"Mom Laptop"		<network> (gWZ, gNetworkMom)
    Switch landevice_MomPhone		"Mom Phone"		<network> (gWZ, gMOM_PRESENT, gNetworkMom, gPhones)
    Switch landevice_MomLaptopWork	        "Mom Laptop@work"	<network> (gWZ, gNETWORKMom)
    Switch landevice_MomPhoneWork	        "Mom Phone@work"	<network> (gWZ, gMOM_PRESENT, gNetworkMom, gPhones)
    Switch landevice_GuestLaptop	        "Guest Laptop"		<network> (DG_GUEST, gNetworkGuest)
    Switch landevice_GuestPhone		"Guest Phone"		<network> (DG_GUEST, gGUEST_PRESENT, gNetworkGuest, gPhones)
    Switch landevice_GirlLaptop		"Girl Laptop"		<network> (DG_GIRL, gNetworkGirl)
    Switch landevice_GirlPhone		"Girl Phone"		<network> (DG_GIRL, gGIRL_PRESENT, gNetworkGirl, gPhones)
    Switch landevice_BoyLaptop		"Boy Laptop"		<network> (DG_BOY, gNetworkBoy)
    Switch landevice_BoyPhone		"Boy Phone"		<network> (DG_BOY, gBOY_PRESENT, gNetworkBoy, gPhones)
    Switch landevice_BoyiPod		"Boy iPod"		<network> (DG_BOY, gNetworkBoy)

As you can see the devices are assigned to various groups in the config, the interesting ones are the "_PRESENT" ones for the phones. Based on these with rules the status of each person's presence in the house can be defined. In my case when at least one phone per kid/guest is active in the network this person is seen as "present", for mom and dad both devices have to be present (on the weekends the work phones usually stay connected at home). Adding all devices here also offers some other capabilities... For example lower the download speed of the download server based on the number of active laptops in case the internet connection is not lighting fast.

#### Final steps on the Linux server

Another test: Now execute one of the *curl* statements created in the previous step on the Linux server in the shell:

    ReadyNAS:~# curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_KGTEC2AP -d "ON"
    ReadyNAS:~#

If the items were created correctly this should generate no output at all in the Linux shell, but show up promptly in the openHAB event log:

    ReadyNAS:~# tail -200 /opt/openhab/logs/events.log
    ...
    2013-11-13 22:47:25 - landevice_KGTEC2AP received command ON
    ...

Now create the final script _fritzbox_devices.sh_ on the Linux server

    #!/bin/sh
    /root/fritzbox_devices.expect | /bin/fgrep "landevice=" | /root/fritzbox_devices.sed > /tmp/commands.txt
    . /tmp/commands.txt
    rm /tmp/commands.txt

and test this script again, this time using *time* to avoid too many calls by cron:

    ReadyNAS:~# time ./fritzbox_devices.sh
    real    0m13.637s
    user    0m0.267s
    sys     0m0.202s
    ReadyNAS:~#

Again, you should see no output in case you defined all items in openHAB. In case you did not errors will show up here and in the openHAB logs. A "clean" openHAB event.log outputs looks as follows:

    ReadyNAS:~# tail -200f /opt/openhab/logs/events.log
    ...
    2013-11-13 22:58:52 - landevice_BoyLaptop received command OFF
    2013-11-13 22:58:52 - landevice_BoyPhone received command ON
    2013-11-13 22:58:52 - landevice_BoyiPod received command OFF
    2013-11-13 22:58:52 - landevice_DGOfficePrinter received command ON
    2013-11-13 22:58:52 - landevice_DGTVRasPlex received command ON
    2013-11-13 22:58:52 - landevice_DGTVWii received command OFF
    2013-11-13 22:58:52 - landevice_DadLaptop received command OFF
    2013-11-13 22:58:52 - landevice_DadLaptopWork received command OFF
    2013-11-13 22:58:52 - landevice_DadLaptopWork received command OFF
    2013-11-13 22:58:52 - landevice_DadPC received command ON
    2013-11-13 22:58:52 - landevice_DadPhone received command ON
    2013-11-13 22:58:52 - landevice_DadPhoneWork received command ON
    2013-11-13 22:58:52 - landevice_EGEntranceAP received command ON
    2013-11-13 22:58:52 - landevice_EGTVRasPlex received command ON
    2013-11-13 22:58:52 - landevice_EGWZAP received command OFF
    2013-11-13 22:58:52 - landevice_GirlLaptop received command OFF
    2013-11-13 22:58:52 - landevice_GirlPhone received command ON
    2013-11-13 22:58:52 - landevice_GuestLaptop received command OFF
    2013-11-13 22:58:52 - landevice_KGOfficePrinter received command ON
    2013-11-13 22:58:52 - landevice_KGOfficeReadyNAS received command ON
    2013-11-13 22:58:52 - landevice_KGTEC1AP received command ON
    2013-11-13 22:58:52 - landevice_KGTEC2AP received command ON
    2013-11-13 22:58:52 - landevice_MomLaptop received command OFF
    2013-11-13 22:58:52 - landevice_MomLaptop received command OFF
    2013-11-13 22:58:52 - landevice_MomLaptopWork received command OFF
    2013-11-13 22:58:52 - landevice_MomPhone received command OFF
    ...

When everything is working fine the cronjob can be set up, but avoid calling the script to frequently (thus the *time* before). In this case we set it up to run each minute:

    # m h  dom mon dow   command
    * * * * * /root/fritzbox_devices.sh

#### (Yet) incomplete snipplets with additional functionalities for the Fritz!Box

*wlandevices.sh* is just dumping all devices connected directly to the wlan of the Fritz!Box (not those attached through LAN ports, but including the guest WLAN). For example to show devices connected to the guest WLAN with their remaining time in the openHAB GUI:

    #!/bin/sh
    i=0
    count_devices=`ctlmgr_ctl r wlan settings/wlanlist/count`
    while [ $i -lt $count_devices ]
    do
    device="wlandevice="$i
    #for command in mac UID state speed rssi quality is_turbo is_guest is_ap ap_state mode wmm_active cipher powersave is_repeater channel freq flags flags_set
    for command in mac state speed is_guest
    do
    output="`ctlmgr_ctl r wlan settings/wlanlist$i/$command`"
    device=$device" "$command"="$output
    done
    echo $device
    i=`expr $i + 1`
    done

The output looks as follows and again can be adopted using the commented out for loop:

    # ./wlandevices.sh
    wlandevice=0 mac=AC:81:12:B0:39:F0 state=0 speed=0 is_guest=0
    wlandevice=1 mac=20:02:AF:89:59:29 state=0 speed=0 is_guest=0
    wlandevice=2 mac=18:E7:F4:09:A7:E4 state=0 speed=0 is_guest=0
    wlandevice=3 mac=00:1F:1F:A2:AE:A6 state=5 speed=97 is_guest=0
    wlandevice=4 mac=00:27:09:EA:D3:E9 state=0 speed=0 is_guest=0
    wlandevice=5 mac=00:1C:B3:C3:5F:B3 state=0 speed=0 is_guest=0
    wlandevice=6 mac=00:27:10:4F:3F:F0 state=0 speed=0 is_guest=0
    wlandevice=7 mac=98:FE:94:39:AC:49 state=5 speed=41 is_guest=0
    wlandevice=8 mac=54:44:08:A9:C4:F4 state=5 speed=106 is_guest=0
    wlandevice=9 mac=80:1F:02:63:D3:32 state=5 speed=113 is_guest=0
    wlandevice=10 mac=EC:55:F9:19:40:43 state=0 speed=0 is_guest=0
    wlandevice=11 mac=64:66:B3:4B:A5:AE state=5 speed=65 is_guest=0
    wlandevice=12 mac=64:66:B3:4B:C3:46 state=5 speed=38 is_guest=0
    wlandevice=13 mac=B4:B6:76:AD:22:4E state=0 speed=0 is_guest=0
    wlandevice=14 mac=58:1F:AA:AD:A6:E1 state=0 speed=0 is_guest=0
    wlandevice=15 mac=00:80:92:97:3E:AB state=5 speed=64 is_guest=0

When you want to refer to device names, the mac addresses need to be added to the output of *landevices.sh* above and the output of both scripts needs to be matched by mac address which should be easily possible. 

Furthermore *wlanstatus.sh* just dumps the status of the wireless networks of the Fritz!Box:

    #!/bin/sh
    for wlan in "" guest_
    do
    device=$wlan"wlan: "
    for command in ap_enabled ssid pskvalue time_remain
    do
    output="`ctlmgr_ctl r wlan status/$wlan$command`"
    device=$device" "$command"="$output
    done
    echo $device
    done

The output looks as follows:

    # ./wlanstatus.sh
    wlan: ap_enabled=1 ssid=wlanssid pskvalue=unencryptedpassword time_remain=
    guest_wlan: ap_enabled=0 ssid=guestssid pskvalue=unencryptedpassword time_remain=0

This script might be used together with another (yet to do) guest wlan on/off script to offer kids the possibility to enable/disable the guest wlan with a randomly generated password (eg  *echo `tr -dc A-Za-z0-9* < /dev/urandom | head -c 16`_) in order to not have to do this yourself each time.


### Use Fritzbox SIP Feature to make a call in case of an event


**Prologue:**
This Guide shows you how to make a call with a sip-client over a fritz box.
But it should work with all other sip gateways as well – give it a try!

**Requirements:**
- AVM Fritzbox (tested with 7490)
- openhab (obviously)
- soundcard (dummy is ok if you don’t have one. Give ALSA a try)
- sipcmd
- a couple of wav-files (8kHz, 16 bit per Sample, PCM)
- the power of Copy and Paste ;-)

First of all, add a new ip phone to your fritzbox by doing this:

_(note: I only got a german Version of FritzOS: which means its freely translated to English by me; don’t hit me if says something else on your box :-)_

1. Logon to your fritz.box
2. On the left, click on "telephony" (GER: Telefonie)
3. Next, click on "devices" (GER: Telefonie-Geräte)
4. On the right site at the bottom, click on "Create a new device" (GER: Neues Gerät einrichten)
5. Now select “Phone” (GER: Telefon (mit und ohne Anrufbeantworter)) and click next.
6. Select "LAN/WiFi (IP-Phone))" (GER: LAN/WLAN (IP-Telefon)) and enter a name for this device. Click next to continue.
7. Enter a Username (mostly a 3 Digit value; the internal Phone number) and enter a password. Click next.
8. Now select the outgoing caller-id and carry on.
9. Next, select "accept only calls for the following numbers" and deselect all numbers. Almost there.
10. Last step: apply the Settings. DONE! Congrats!

**Moving on to SIPCMD:**

Did I mention that you need a sound card? Don’t got one? Pah, no probs. ALSA (http://www.alsa-project.org/main/index.php/Main_Page) got your back. Under Debian just install alsa-base "apt-get install alsa-base".


1. Download the SIPCMD Branch:  
     `wget https://github.com/tmakkonen/sipcmd/archive/master.zip`
1. Unzip it. Go into the sipcmd-master directory  
   needed Packages:  
    `apt-get install libopal-dev libpt-dev`  
    (more infos: https://github.com/tmakkonen/sipcmd)    
1. Now just run “make”  
1. Copy the sipcmd. To your preferd BIN Dir (/bin or /usr/bin)  
1. That’s it.

Now go and Record some Wav-Files! You’ll need them later… you also could use text-to-speech engines to get some wav-files – be free. 

Put them in this Directory: /opt/openhab/etc/sipcalls/

Create a "Script" with your favourite Text Editor (like nano, vi and so on) for each Event you want to cover. 


**Example:**

    /opt/openhab/etc/sipcalls/sipcall_alarm_window_bathroom.sh

  Now put this line in:

     sipcmd -u 621 -c verysecurepassord -P sip -w fritz.box -x "c012345678;ws500;vwindow_bathroom.wav;h"

_(If you want to call another number, just copy and paste the line above as often as you want.)_

  **Explanation:**

    -u is the username – mostly a 3 digit number (see Point 7. From the fritzbox setup)
    -c is the password you enterd earlier
    -P is the Protocol which is used (SIP)
    -w is the SIP-Domain; in this case fritz.box
    -x the commad that sipcmd should execute where:
    C= is the number which should be called
    ws= waittime in milliseconds: meaning wait for x millis after the remote party answerd the call before playing the wav-file
    v= filename of the wav-file
    H= hangup after the file played
   
Now we just need to define an Item (in /opt/openhab/configurations/items/*.items )

      Switch   SIPCALL_WINDOW_BATHROOM   "SipCall Window Bathroom INTERNAL"   { exec="ON:/opt/openhab/etc/sipcalls/sipcall_alarm_window_bathroom.sh "}

Finally, create a Rule which uses the item:

      Rule "Window_Bathroom_opened"
      when 
      Item WINDOW_BATHROOM received update OPEN
      then
      If (ALARMSYSTEM_ACTIVE.state == ON)
      {
      sendCommand( SIPCALL_WINDOW_BATHROOM, ON)
      // the sendMail command is just another way to get notified
      sendMail("mymail@adresss.com", "OpenHab: Window Bathroom opened!", "please check!")
      }
      End

Thats it.

### How to configure openHAB to connect to device symlinks (on Linux)

When connecting serial devices to Linux (i.e. USB) they are assigned arbitrary device names - usually something like /dev/ttyUSB0. There is no way to know exactly what name will be assigned each time you plug in the device however.

Using udev rules you can setup symlinks which are created when the system detects a device is added. The symlink allows you to assign a 'known' name for that device. See this link for a useful tutorial about udev - http://www.reactivated.net/writing_udev_rules.html. 

So for example, if you have a ZWave USB dongle you can configure a symlink called /dev/zwave. This makes it much easier to configure connection properties since this name will never change.

However the RXTX serial connection library used in openHAB is unable to 'see' these symlinks. Therefore you need to inform RXTX about your symlinks using a system property.

You can either add the property to the Java command line by adding the following (device names delimited by :

`-Dgnu.io.rxtx.SerialPorts=/dev/rfxcom:/dev/zwave`

Or add a gnu.io.rxtx.properties file which is accessible in the Java classpath. See http://create-lab-commons.googlecode.com/svn/trunk/java/lib/rxtx/README.txt for more details.

This applies to any openHAB binding that uses the RXTX serial connection library - i.e. RFXCOM, ZWave etc.

### Use URL to manipulate items

When you have a device unable to send REST requests (f.e. Webcams), you may use

    http://<openhab-host>:8080/CMD?<item>=<state>

Example:

    http://<openhab-host>:8080/CMD?Light=ON
    http://<openhab-host>:8080/CMD?Light=OFF
    http://<openhab-host>:8080/CMD?Light=TOGGLE

TOGGLE changes the current status of a switch item from ON to OFF or from OFF to ON.
## Extract caller and called number from Fritzbox Call object

If you define an item like the following in your site.items config

    Call   Incoming_Call_No   "Caller No. [%2$s]"   (Phone)   { fritzbox="inbound" }

the Type CallType with its associated methods can be used to extract the caller and called number of the call. The following rule sends an email with both numbers in the subject line.

    import org.openhab.library.tel.types.CallType
    
    rule "inbound Call"
      when Item Incoming_Call_No received update
    then 
      var CallType call = Incoming_Call_No.state as CallType
      var String mailSubject =
      "Anruf von Nummer " + call.origNum + " -> " + call.destNum
      sendMail( "us...@domain.de" , mailSubject , "");
    end
   
### Item loops with delay

Some integrations don't like to be rushed with a cascade of simultaneous commands. This can be an issue if you, for instance want to implement a rule which loops through a list of group members (items) and send a command to all of them, e. g. OFF. The solution is to add a small delay between each command. The example below executes OFF for all members at a one second interval. Use `now.plusMillis(i*`*n*`)` for *n* millisecond intervals.

    rule "Lights out"
    when 
    	Time cron "0 0 22 ? * MON-THU,SUN *" or
    	Time cron "0 59 23 ? * FRI,SAT *"
    then 
    	gLightOffNight?.members.forEach(item,i|createTimer(now.plusSeconds(i)) [|sendCommand(item, OFF)])
    end

### enocean binding on Synology DS213+ (kernel driver package)
The enocean binding works with the [enocean USB300 usb-stick](http://www.enocean.com/de/enocean_module/usb-300-oem/).
To enable the USB300 on a [Synology DS213+](https://www.synology.com/en-us/products/overview/DS213+) (probably works also on Synology DS413 - same CPU...) two things are needed:

1. The kernel drivers  
usbserial, ftdi_sio and cdc-acm are already included in DSM 4.3  
pl2303 and cp210x are in the package usb-driver-kernel-2.6.32-dh-syno-qoriq-0.001.spk  
The package is stored at [OpenHAB google groups](https://groups.google.com/d/msg/openhab/tF9Y3wOLYX8/fOlU2lD0q2UJ) and on the package server [https://www.hofrichter.at/sspks](https://www.hofrichter.at/sspks/index.php?fulllist=true)  
You can install this package in DSM via the package manager -> manual installation or by adding https://www.hofrichter.at/sspks/ to your package sources - there is a [tutorial on the Synology support pages](http://www.synology.com/en-us/support/tutorials/500) about how to do that.
2. nrjavaserial  
OpenHab 1.3.1 comes with the latest stable version of [nrjavaserial 3.8.8]
(https://code.google.com/p/nrjavaserial/) (at the moment (17.11.2013)).  
It's in {openhab_root}/server/plugins/org.openhab.io.transport.serial  
The newer **unstable** nrjavaserial **3.9.1** is needed for the qoric cpu of the DS213+ and included in the (otherwise unchanged) [org.openhab.io.transport.serial_1.3.1.201309182025.jar](https://www.hofrichter.at/sspks/packages/org.openhab.io.transport.serial_1.3.1.201309182025.jar).  
Just put that file in {openhab_root}/server/plugins/ and delete the original file {openhab_root}/server/plugins/org.openhab.io.transport.serial...

### How to use openHAB to activate or deactivate your Fritz!Box-WLAN

The Fritz!Box-Binding already supports controlling the built-in WIFI/WLAN and DECT. Please see the [[Binding page|Fritz-Box-Binding#switching-wifi-and-dect]].


### How to display Google Maps in a sitemap from a Mqttitude Mqtt message

First, create a MQTT entry in your items file that subscribes to the Mqttitude push:

`String 	Location_Dan_Phone {mqtt="<[home:owntracks/daniel/iphone5s:state:JS(mqttitude-coordinates.js)]"}`

The transform is a javascript file in the configuration/Transform directory (mqttitude-coordinates.js).  This file is very simple:
```
var location = eval('(' + input + ')');
result = location.lat + "," + location.lon;
```
The value of your item (Location_Dan_Phone) will be set to the result value. 

Create a webpage that you will put in the webapps/static directory, call it map.html
```
<!DOCTYPE html>
<html>
  <head>
    <style>
      #map_canvas {
        width: 900px;
        height:500px;
      }
    </style>
    <script src="//ajax.googleapis.com/ajax/libs/jquery/2.1.0/jquery.min.js"></script>
    <script src="//maps.googleapis.com/maps/api/js?sensor=false"></script>
    <script>
      $(function() {
	$.ajax({
		url: "/rest/items/Location_Dan_Phone?type=json",
		dataType: "json",
  		data: {
		},
  		success: function( data ) {
			 console.log(data);
			 var coords = data.state.split(',');
			 var latlng = new google.maps.LatLng(coords[0],coords[1]);
			 var map_canvas = document.getElementById('map_canvas');
		         var map_options = {
          			center: latlng,
          			zoom: 14,
          			mapTypeId: google.maps.MapTypeId.HYBRID
        		}
        		var map = new google.maps.Map(map_canvas, map_options)
			var marker = new google.maps.Marker({
				position: latlng,
    				map: map,
    				title:"Dan's Phone",
				image: "http://s1.hubimg.com/u/7148626_f520.jpg"
			});
      		}
  	  });
	});
    </script>
  </head>
  <body>
    <div id="map_canvas"></div>
  </body>
</html>
```
now reference the full URL to this page in a webview widget in your sitemap, something like
```
Webview  url="https://openhab.mydomain.com:8443/static/map.html"  height=12
```
Make sure you you have username and password authentication turned on!

### How to use Yahoo weather images

This tip allows you to use Yahoo! weather images in your sitemap. See the [Yahoo! Weather Developer]( http://developer.yahoo.com/weather/) page for more information.

First we need to download a current set of yahoo images to the runtime/webapps/images folder, there are 48 total.  This command assumes you have imagemagick installed. This is included in most linux distros, on mac you can install it using [brew](http://brew.sh/)  like : ` brew install imagemagick `.

```
for i in `seq 0 47`;do curl http://l.yimg.com/a/i/us/we/52/$i.gif | convert - yahoo_weather-$i.png ;done
```

Note that we prefix the images with yahoo_weather, this is important as openHAB will append a yahoo weather code later to choose the correct one.

Next add a xsl transform file to your configuration/Transform directory, name this yahoo_weather_code.xsl
```
<?xml version="1.0"?>
<xsl:stylesheet 
	xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
	xmlns:yweather="http://xml.weather.yahoo.com/ns/rss/1.0" version="1.0">
	<xsl:output indent="yes" method="xml" encoding="UTF-8" omit-xml-declaration="yes" />
	<xsl:template match="/">
		<!--<xsl:text>yahoo-weather-</xsl:text>-->
		<xsl:value-of select="//item/yweather:condition/@code" />
	</xsl:template>
</xsl:stylesheet>
```

This will return the Yahoo! weather code for a selected region.

We can map the weather code to a description so the item has both an image and text.  Name this yahoo_weather_code.map in the Configuration/Transform directory.
```
0=tornado
1=tropical storm
2=hurricane
3=severe thunderstorms
4=thunderstorms
5=mixed rain and snow
6=mixed rain and sleet
7=mixed snow and sleet
8=freezing drizzle
9=drizzle
10=freezing rain
11=showers
12=showers
13=snow flurries
14=light snow showers
15=blowing snow
16=snow
17=hail
18=sleet
19=dust
20=foggy
21=haze
22=smoky
23=blustery
24=windy
25=cold
26=cloudy
27=mostly cloudy (night)
28=mostly cloudy (day)
29=partly cloudy (night)
30=partly cloudy (day)
31=clear (night)
32=sunny
33=fair (night)
34=fair (day)
35=mixed rain and hail
36=hot
37=isolated thunderstorms
38=scattered thunderstorms
39=scattered thunderstorms
40=scattered showers
41=heavy snow
42=scattered snow showers
43=heavy snow
44=partly cloudy
45=thundershowers
46=snow showers
47=isolated thundershowers
```

Add a Number item to a items file

```
Number YahooWeatherCode "Today is [MAP(yahoo_weather_code.map):%s]" (weather) { http="<[http://weather.yahooapis.com/forecastrss?w=2459115:3600000:XSLT(yahoo_weather_code.xsl)]"}
```

And a Text item to your sitemap

```
Text item=YahooWeatherCode icon="yahoo_weather"
```


### How to wake up with Philips Hue

You want to make your morning light up with philips hue, the follow the steps

First you make a rule to wake up a certain time, for example 6:30 in the morning between Monday and Friday.
This rule call the script dimmlight wich you can use later also in a other rule.

```
rule "Wakeup"

when 
	    Time cron "0 30 06 ? * MON-FRI "  
then


callScript("dimmlight")

end
```
Now you make the script dimmlight.script
```
var Number Dimmer

Dimmer=0


 
while(Dimmer<100 )


{
        Dimmer=Dimmer+5
        sendCommand(Light_LR_Living_Sofa_D,Dimmer)
        Thread::sleep(400)
        
}
```

```
Thread::sleep(400) 
```
a higher value will dimm the light slower, a lower value will dimm the light faster

### How to manage and sync configuration via subversion

In this guide subversion is used to both store your config and sync it between you local computer and the openhab server.

#### Setting up subversion
First setup a subversion server on your openhab server. You'll will find plenty of howtos how to do that.
We asume that your repo is stored in /var/lib/svn/openhab and is running under the system user www-data

#### Install hook script
A hook script is used to apply all changes in the subversion repo to the actual openhab config.
Create a hook-script /var/lib/svn/openhab/hooks/commit-hook with the following contents

```
#!/bin/sh
REPOS="$1"
REV="$2"
svn up /opt/openhab/configurations
```

The path in the last line has to match to your openhab installation

#### Import your config into subversion

Probably you already have an configuration. Import this into your subversion server. On Windows TortoiseSVN does a good job for that.

#### Checkout openhab config on server

You have to checkout the configuration into openhab once

`svn co http://yourhost/svn/openhab /opt/openhab/configurations`

Maybe you have to delete your existing configuration folder in /opt/openhab first

#### Setting up permissions

The tricky part is to setup the permissions right. I assume you subversion server runs with www-data and openhab with openhab system user. The problem is that www-data needs the permission to write to your openhab configuration.

`chown -R openhab.www-data /opt/openhab/configuration`
`find /opt/openhab/configuration -type d -exec chmod g+ws {} \;`

### How to switch LEDS on cubietruck

#### Items

```
/* LEDS */
Switch LED_Blue	{exec=">[ON:/bin/sh@@-c@@echo 1 | sudo tee /sys/class/leds/blue:ph21:led1/brightness] >[OFF:/bin/sh@@-c@@echo 0 | sudo tee /sys/class/leds/blue:ph21:led1/brightness]"}
Switch LED_Orange {exec=">[ON:/bin/sh@@-c@@echo 1 | sudo tee /sys/class/leds/orange:ph20:led2/brightness] >[OFF:/bin/sh@@-c@@echo 0 | sudo tee /sys/class/leds/orange:ph20:led2/brightness]"}
Switch LED_White {exec=">[ON:/bin/sh@@-c@@echo 1 | sudo tee /sys/class/leds/white:ph11:led3/brightness] >[OFF:/bin/sh@@-c@@echo 0 | sudo tee /sys/class/leds/white:ph11:led3/brightness]"}
Switch LED_Green {exec=">[ON:/bin/sh@@-c@@echo 1 | sudo tee /sys/class/leds/green:ph07:led4/brightness] >[OFF:/bin/sh@@-c@@echo 0 | sudo tee /sys/class/leds/green:ph07:led4/brightness]"}
```

#### Permissions
Create `/etc/sudoers.d/leds`

```
ALL ALL = (ALL) NOPASSWD: /usr/bin/tee /sys/class/leds/blue\:ph21\:led1/brightness
ALL ALL = (ALL) NOPASSWD: /usr/bin/tee /sys/class/leds/green\:ph07\:led4/brightness
ALL ALL = (ALL) NOPASSWD: /usr/bin/tee /sys/class/leds/white\:ph11\:led3/brightness
ALL ALL = (ALL) NOPASSWD: /usr/bin/tee /sys/class/leds/orange\:ph20\:led2/brightness
```

### How to use a voice command from HABDroid

HABDroid uses a predefined string item to pass voice commands - the name of this item is "VoiceCommand".
So all you have to do is to add such an item in your *.items file:

```
String VoiceCommand
```

And react on commands for this item in a rule:

```
 rule VoiceControl
 when
 	Item VoiceCommand received command
 then
 	val hue = switch(receivedCommand.toString.lowerCase) {
	 	case "blue"    : 240
	 	case "magenta" : 300
	 	case "green"   : 120
	 	case "yellow"  : 70
 	}
 	Light.sendCommand(hue + ",100,100")
 end
```
A rule, which reacts flexible on natural spoken sentences:
```
rule "Voice control"
when
    Item VoiceCommand received command
then
    val txt = receivedCommand.toString.toLowerCase
//Wohnzimmer
    if(txt.contains("front") || txt.contains("fernseh")) {
        if(txt.contains("ein")) sendCommand(WandlichtWohnzimmerFront, ON) else 
        if(txt.contains("start")) sendCommand(WandlichtWohnzimmerFront, INCREASE) else
        if(txt.contains("an")) sendCommand(WandlichtWohnzimmerFront, ON) else
        if(txt.contains("aus")) sendCommand(WandlichtWohnzimmerFront, OFF) else
        if(txt.contains("stop")) sendCommand(WandlichtWohnzimmerFront, STOP) else
        if(txt.contains("heller")) sendCommand(WandlichtWohnzimmerFront, INCREASE) else
        if(txt.contains("dunkler")) sendCommand(WandlichtWohnzimmerFront, DECREASE) 
    }  else if(txt.contains("decke")) {
        if(txt.contains("ein")) sendCommand(DeckenlichtWohnzimmerSchalten, ON) else
        if(txt.contains("start")) sendCommand(DeckenlichtWohnzimmerSchalten, ON) else
        if(txt.contains("an")) sendCommand(DeckenlichtWohnzimmerSchalten, ON) else
        if(txt.contains("aus")) sendCommand(DeckenlichtWohnzimmerSchalten, OFF) else
        if(txt.contains("stop")) sendCommand(DeckenlichtWohnzimmerSchalten, OFF)

//Küche
    } else if(txt.contains("küche")) {
        if(txt.contains("ambiente") || txt.contains("boden")) {
            if(txt.contains("ein")) sendCommand(LEDKuecheSchalten, ON) else
            if(txt.contains("start")) sendCommand(LEDKuecheSchalten, ON) else
            if(txt.contains("an")) sendCommand(LEDKuecheSchalten, ON) else
            if(txt.contains("aus")) sendCommand(LEDKuecheSchalten, OFF) else
            if(txt.contains("stop")) sendCommand(LEDKuecheSchalten, OFF)
        } else if(txt.contains("licht") || txt.contains("decke") || txt.contains("leuchte")) {
            if(txt.contains("ein")) sendCommand(DeckenlichtKueche, ON) else
            if(txt.contains("start")) sendCommand(DeckenlichtKueche, ON) else
            if(txt.contains("an")) sendCommand(DeckenlichtKueche, ON) else
            if(txt.contains("aus")) sendCommand(DeckenlichtKueche, OFF) else
            if(txt.contains("stop")) sendCommand(DeckenlichtKueche, OFF)
        }
//Bad
    } else if(txt.contains("bad")) {
        if(txt.contains("wanne")) {
            if(txt.contains("ein") || txt.contains("start") || txt.contains("an")) sendCommand(WannenlichtBad, ON) else
            if(txt.contains("aus") || txt.contains("stop")) sendCommand(WannenlichtBad, OFF) 
        } else if(txt.contains("decke")) {
            if(txt.contains("ein") || txt.contains("start") || txt.contains("an")) sendCommand(DeckenlichtBad, ON) else
            if(txt.contains("aus") || txt.contains("stop")) sendCommand(DeckenlichtBad, OFF) 
        }
//Galerie
    }  else if(txt.contains("galerie")) {
        if(txt.contains("licht")) {
            if(txt.contains("ein")) sendCommand(DeckenlichtGalerie, ON) else
            if(txt.contains("start")) sendCommand(DeckenlichtGalerie, INCREASE) else
            if(txt.contains("an")) sendCommand(DeckenlichtGalerie, ON) else
            if(txt.contains("aus")) sendCommand(DeckenlichtGalerie, OFF) else
            if(txt.contains("stop")) sendCommand(DeckenlichtGalerie, STOP) else
            if(txt.contains("heller")) sendCommand(DeckenlichtGalerie, INCREASE) else
            if(txt.contains("dunkler")) sendCommand(DeckenlichtGalerie, DECREASE) 
        } else if(txt.contains("drucker")) {
            if(txt.contains("ein")) sendCommand(DruckerSchalten, ON) else
            if(txt.contains("an")) sendCommand(DruckerSchalten, ON) else
            if(txt.contains("aus")) sendCommand(DruckerSchalten, OFF) 
        }

//Rollo    
    } else if(txt.contains("rollo") || txt.contains("jalousie")) {
        if(txt.contains("hoch") || txt.contains("auf") || txt.contains("öffnen")) {
            if(txt.contains("wohnzimmer")) sendCommand(RolloWZgemeinsam, UP) else
            if(txt.contains("bad")) sendCommand(RolloBad, UP) else
            if(txt.contains("küche")) sendCommand(RolloKueche, UP) else
            if(txt.contains("galerie")) sendCommand(RolloGalerie, UP) else {
                sendCommand(RolloWZgemeinsam, UP)
                sendCommand(RolloSZBADGZgemeinsam, UP)
            }
        } else if(txt.contains("runter") || txt.contains("ab") || txt.contains("schließen")) {
            if(txt.contains("wohnzimmer")) sendCommand(RolloWZgemeinsam, DOWN) else
            if(txt.contains("bad")) sendCommand(RolloBad, DOWN) else
            if(txt.contains("küche")) sendCommand(RolloKueche, DOWN) else
            if(txt.contains("galerie")) sendCommand(RolloGalerie, DOWN) else {
                sendCommand(RolloWZgemeinsam, DOWN)
                sendCommand(RolloSZBADGZgemeinsam, DOWN)
            }
        }  else if(txt.contains("stop")) {
            if(txt.contains("wohnzimmer")) sendCommand(RolloWZgemeinsam, STOP) else
            if(txt.contains("bad")) sendCommand(RolloBad, STOP) else
            if(txt.contains("küche")) sendCommand(RolloKueche, STOP) else
            if(txt.contains("galerie")) sendCommand(RolloGalerie, STOP) else {
                sendCommand(RolloWZgemeinsam, STOP)
                sendCommand(RolloSZBADGZgemeinsam, STOP)
            }
        }
    } else if(txt.contains("markise")) {
        if(txt.contains("hoch") || txt.contains("auf") || txt.contains("öffnen")) sendCommand(Markise, UP) else
        if(txt.contains("runter") || txt.contains("ab") || txt.contains("schließen")) sendCommand(Markise, DOWN) else
        if(txt.contains("stop")) sendCommand(Markise, STOP)
            
    //Musik    
    } else if(txt.contains("musik")) {
        if(txt.contains("ein") || txt.contains("an") ||txt.contains("start")) {
            if(txt.contains("wohnzimmer")) sendCommand(sq_wohnen_power, ON) else
            if(txt.contains("gästezimmer")) sendCommand(sq_gaeste_power, ON) else
            if(txt.contains("schlafzimmer")) sendCommand(sq_schlafen_power, ON) else
            if(txt.contains("galerie")) sendCommand(sq_galerie_power, ON) else
            if(txt.contains("bad")) sendCommand(MusikBadSchalten, ON) else
//            if(txt.contains("wc")) sendCommand(MusikWCSchalten, ON) else
            if(txt.contains("balkon")) sendCommand(MusikBalkonSchalten, ON) else {
                sendCommand(sq_wohnen_power, ON)
            }
 
        } else if(txt.contains("aus") || txt.contains("stop")) {
            if(txt.contains("wohnzimmer")) sendCommand(sq_wohnen_power, OFF) else
            if(txt.contains("gästezimmer")) sendCommand(sq_gaeste_power, OFF) else
            if(txt.contains("schlafzimmer")) sendCommand(sq_schlafen_power, OFF) else
            if(txt.contains("galerie")) sendCommand(sq_galerie_power, OFF) else
            if(txt.contains("bad")) sendCommand(MusikBadSchalten, OFF) else
//            if(txt.contains("wc")) sendCommand(MusikWCSchalten, OFF) else
            if(txt.contains("balkon")) sendCommand(MusikBalkonSchalten, OFF) else {
                sendCommand(sq_wohnen_power, OFF)
                sendCommand(sq_schlafen_power, OFF)
            }
 
        }
   
end
```
### Add Humidex calculation for your Feels Like Temperature value

I think we can do the same with Heat index for US, but for the moment this is the european version, it works perfectly with Netatmo but it should be ok for any weather/climate technology.

The items file could look like this:

```
/*Humidex "Feels like" Temperature */
Number Humidex_Outdoor_Temperature "Feels like [%.1f °C]" <temp_windchill>	(Weather)
```

The rules file looks like this:

```
// Humidex Rule
rule "Humidex calculation"
when
  Item Netatmo_Outdoor_Temperature changed or
  Item Netatmo_Outdoor_Humidity changed or
  System started
then
  var Number T = Netatmo_Outdoor_Temperature.state as DecimalType
  var Number H = Netatmo_Outdoor_Humidity.state as DecimalType
  var Number x = 7.5 * T/(237.7 + T)
  var Number e = 6.112 * Math::pow(10, x.doubleValue) * H/100
  var Number humidex = T + (new Double(5) / new Double(9)) * (e - 10)
  postUpdate(Humidex_Outdoor_Temperature, humidex);
end
```
### Aeon Zstick Setup in Linux
The Aeon Zstick is a serial USB device in Linux, but it does not use the generic usbserial device drivers. 

Running lsusb you will find it is a CP210x device:
```
#lsusb
Bus 001 Device 002: ID 10c4:ea60 Cygnal Integrated Products, Inc. CP210x Composite Device
```
To get it working you will need to be sure that you have P210x serial device drivers.
```
# dmesg | grep cp210x
cp210x: v0.08:Silicon Labs CP2101/CP2102 RS232 serial adaptor driver
```
If not you will need to find and install the package for your distro.  Once that is done you should be able to verify your USB device in the dmesg logs:
```
Aug 16 15:59:59 (none) user.info kernel: USB Serial support registered for cp2101
Aug 16 15:59:59 (none) user.info kernel: cp2101 1-1:1.0: cp2101 converter detected
Aug 16 15:59:59 (none) user.info kernel: usb 1-1: reset full speed USB device using uhci_hcd and address 2
Aug 16 15:59:59 (none) user.info kernel: usb 1-1: cp2101 converter now attached to ttyUSB0
Aug 16 15:59:59 (none) user.info kernel: usbcore: registered new interface driver cp2101
Aug 16 15:59:59 (none) user.info kernel: cp210x: v0.08:Silicon Labs CP2101/CP2102 RS232 serial adaptor driver
```
Most full Linux distro's will have the CP210x module by default, but I am running SLiTaZ linux which is a very lo-ram slimmed down distro (runs in 30MB) so I provide these instructions for anyone else wondering why the device might not be working out of the box in their environment


### How to use a serial port under linux

If you are usiung a usb-to-serial-adapter your serial-interface might be /dev/ttyUSB0.

Check the permissions of this device with

    user@computer:~$ ls -l /dev/ttyUSB0
    crw-rw---- 1 root dialout 188, 0 Aug 11 14:16 /dev/ttyUSB0
In order to access this device the user running openHAB needs to be in the group "dailout" and possibly the group "lock".

    sudo adduser <user> dialout
    sudo adduser <user> lock

Reboot your system.

### Using the transceiver RFXtrx433E with Somfy RTS devices

The first thing you have to do is to pair the RFXtrx433E transceiver with your Somfy RTS devices (see http://rfxcom.com/Documents/RFXtrx%20User%20Guide.pdf for more details). 
If you pair for example a roller shutter with the ID 1.01.01.1, then the binding should look like:

```
Rollershutter rs_1 "Yoda" <rollershutter> (Rollershutters) {rfxcom=">1.01.01.1:RFY.RFY:Shutter"}
```

### Talking to a Raspberry ZWAY device with push updates 

(ZWAY version 2.0+ only)

This works for any device on a ZWAY server, in this example we will bind the Alarm V1 event data for a lock to a OpenHAB item.  First create an Item to represent the alarm state, this will use the http binding to get it's initial state and periodically update (every hour) its self in case we loose a message.

```
Number ZWAY_LOCK_ALARM "Last Status [MAP(lock.map):%d]" {http="<[http://raspberry2:8083/ZWaveAPI/Run/devices%%5B2%%5D.Alarm.data.V1event.alarmType.value:3600000:REGEX((\\d*?))]"}
``` 

The URL we are hitting is `raspberry2:8083/ZWaveAPI/Run/devices[2].Alarm.data.V1event.alarmType.value` but in order to pass the '[' and ']' characters we need to escape them with '%%5B' and '%%5D'.

Next we want to be updated when anything changes on the ZWAY device. We are going to load a javascript file that will bind to our device.  In the ZWAY automation UI, make sure you have the Javascript file load module enabled.  Place the following javascript file in "/opt/z-way-server/automation/storage/':
```
this.registerOhBinding = function(zwayName) {
  debugPrint("OH Binding called with name: " + zwayName);

  if (zwayName != "zway") return; // you want to bind to default zway instance

  debugPrint("OH Binding enabled!");

  var devices = global.ZWave["zway"].zway.devices;

  devices[2].Alarm.data.V1event.alarmType.bind(function(){
    debugPrint("Alarm.data.V1event.alarmType changed! " + this.value);
    global.http.request({
      method: 'PUT',
      url: "http://oh.yourdomain.com:8080/rest/items/ZWAY_LOCK_ALARM/state",
      data:this.value + "",
      headers    : { "Content-Type": "text/plain" }
    });
  });
};

debugPrint("Trying to bind OH");
// process all active bindings
if (global.ZWave) {
  global.ZWave().forEach(this.registerOhBinding);
}
// and listen for future ones
global.controller.on("ZWave.register", this.registerOhBinding);
```
Replace the URL in the file to match your OpenHAB instance and item name. In the ZWAY Javascript File load module enter 'storage/zway-oh.js' as the file (or whatever you named it). This code will send a HTTP post to the Item whenever the alarm state changes. 

### How to use the LIFX beta API via executeCommandLine and curl

To control the bulbs via this API you need to send HTTP requests (GET, POST, PUT).
For security reasons you need a authentication token to access your bulbs.
https://github.com/chendo/lifx-http/issues/27#issuecomment-72760322

Once you got the token, you can play around with the API web interface at https://api.lifx.com/

Unfortunately the HTTP binding does not support sending http headers. (Note: this is no longer true -- you _can_ send HTTP headers with the HTTP binding in recent releases.  However, the following is still a useful example.)

The only way to send the request is using the executeCommandLine command and the unix program curl.

A working example looks like this:
```
var String lifxurl   = "https://api.lifx.com:443/v1beta1"
var String lifxtoken = "c7a3****a98c"       // insert your token here

executeCommandLine("curl@@-k@@-H@@Authorization: Bearer " + lifxtoken + "@@-XPOST@@" + lifxurl + "/lights/all/effects/pulse.json?color=green")

// if you want to change the color or state you need to replace "XPOST" with "XPUT"
// according to the documentation on https://api.lifx.com/
executeCommandLine("curl@@-k@@-H@@Authorization: Bearer " + lifxtoken + "@@-XPUT@@" + lifxurl + "/lights/all/color.json?color=hsb:45,1,0.6")
```
Notice the "@@".
This is neccessary, because the executeCommandLine does not support using quotationmarks.
To send the http header as one string, all other spaces need to be replaces by the delimiter "@@".

You may experience some seconds delay, because this is not the best way to implement the LIFX bulbs in openHAB.
There is a dedicated lifx binding for openhab2 wich works without internet (cloud) connection.

### How to turn ON and OFF a Philips Hue with HomeMatic pushbutton

You want to turn ON and OFF a Philips Hue with a physical pushbutton (e.g. HomeMatic HM-PBI-4-FM). You need to define the states of the HomeMatic pushbutton as items in the `.items` file:
```
// Unreachable:
Switch HM_pushbutton_Unreach "Signal Strength [MAP(signal_status.map):%s]" <signal> (g_HM_Signal) {homematic="address=LEQ0012345, channel=0, parameter=UNREACH"}
//LowBattery:
Switch HM_pushbutton_LowBattery "Battery Status [MAP(batterie_status.map):%s]" <signal> (g_HM_Signal) {homematic="address=LEQ0012345, channel=0, parameter=LOWBAT"}
//Channel 1:
Switch HM_pushbutton_Channel_1 "Pushbutton 1" {homematic="address=LEQ0012345, channel=1, parameter=PRESS_SHORT"}
//Channel 1 (Press long):
Switch HM_pushbutton_Channel_1_long "Pushbutton 1 (long)" {homematic="address=LEQ0012345, channel=1, parameter=PRESS_LONG"}
```
Copy these switches for channel 2, 3, and 4 of the HomeMatic pushbutton and add your Hue bulbs like this:
```
Switch Toggle_1   "Hue 1"   (Switching) {hue="1"}
```
See also this [Link](https://github.com/openhab/openhab/wiki/Hue-Binding#examples) for the Hue Color and Dimmer item.
To receive the signals of the pushbutton you have to set the mode of transmission within the CCU/CCU2 to be standard. (enabled security will not work. For more information see [here](https://code.google.com/p/openhab-homematic/wiki/SupportedDevices#Remote_Control))

After this preparation only the rule is missing. Copy the following into your `.rules` file:
```
var Number Toggle_1_on = 0

rule " Channel 1 press short"
when
	Item HM_pushbutton_Channel_1 changed to ON
then
	if (Toggle_1_on == 0){
	sendCommand(Toggle_1, ON)
	Toggle_1_on = 1}
else{
	sendCommand(Toggle_1, OFF)
	Toggle_1_on = 0}
end
```
Pressing the physical pushbutton connected to channel 1 of the HomeMatic HM-PBI-4-FM will now toggle the Philips Hue (Toggle_1) on and off.

### How to create a simple Remote for controlling a Dune HD Player with Http and Denon Bindings

A Dune HD multimedia player (http://dune-hd.com/eng) is easily manageable via Http Binding. The golden rule is to remember that any 4-byte NEC IR code supported by Dune (see http://dune-hd.com/support/rc) needs to be written in the opposite order (e.g. RC button "1" = 00 BF 0B F4 => F40BBF00).

Here a very simple Remote control with some basic commands:

```
/* Dune items */
/* Living */
Switch Dune_Living		"power"     (Dune, Persistence)
Number Dune_RC1_Living  "" 			(Dune)					{autoupdate="false"}
Number Dune_RC2_Living  "" 			(Dune)					{autoupdate="false"}
Number Dune_RC3_Living  "" 			(Dune)					{autoupdate="false"}

rule "Dune Living Room Remote Control line 1"
        when 
                Item Dune_RC1_Living received command 
        then 
                switch(receivedCommand)
                
                {
                    case 1 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=EA15BF00") 

                    case 2 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=E916BF00") 

                    case 3 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=E817BF00") 

                    case 4 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=E718BF00")

                    case 5 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=EB14BF00")
		}
end

rule "Dune Living Room Remote Control line 2"
        when 
                Item Dune_RC2_Living received command 
        then 
                switch(receivedCommand)
                
                {
                    case 1 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=B748BF00") 

                    case 2 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=E11EBF00") 

                    case 3 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=E619BF00") 

                    case 4 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=B649BF00")

                    case 5 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=E21DBF00")
		}
end

rule "Dune Living Room Remote Control line 3"
        when 
                Item Dune_RC3_Living received command 
        then 
                switch(receivedCommand)
    
                {
                    case 1 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=BB44BF00") 

                    case 2 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=AB54BF00") 

                    case 3 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=AF50BF00") 

                    case 4 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=F807BF00")

                    case 5 : sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=FB04BF00")
		}
end
```

And here an example how to integrate with your AVR. In this case, a Denon AVR 3311: e.g. when I click ON I need to switch on the AVR, set the channel to BD (HDMI connected to Dune HD Player) and switch on the Dune Player:

```
rule "Dune Living"
when
	Item Dune_Living received command
then
	if(receivedCommand==ON) {
		// switch on AVR, set input to BD, switch on Dune HD
		sendCommand(DenonPower, ON)
		sendCommand(DenonInputBluray, ON)
		sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=A05FBF00")
	} else if(receivedCommand==OFF) {
		// turn off Dune HD and AVR
		sendHttpGetRequest("http://<ip_address>/cgi-bin/do?cmd=ir_code&ir_code=A15EBF00")
		sendCommand(DenonPower, OFF)
	}
end
```

Then, a very simple sitemap:

```
Text label="Dune HD" icon="dune_hd3"{
	Switch item=Dune_Living icon="control_standby"
	Slider item=DenonVolume icon="audio_audio"
	Text item=DenonVolumedB
	Switch item=DenonMute icon="audio_volume_mute"
	Switch item=Dune_RC1_Living mappings=[1=" Up ", 2=Down, 3=Left, 4=Right, 5=Select]
	Switch item=Dune_RC2_Living mappings=[1=" Play ", 2=Pause, 3=Stop, 4=Prev, 5=Next]
	Switch item=Dune_RC3_Living mappings=[1=" Audio", 2=Subtitle, 3=Info, 4="Pop Up", 5=" Return "]
}
```

Last few notes for DenonVolume, how to see dB and how to start with a predefined level (in this case, 40 dB):

```
Dimmer DenonVolume          "volume [%.1f]"     (Denon)	{denon="avr3311#MV"}
Number DenonVolumedB		"[%.1f dB]"	(Denon)

rule "DENON DB to Volume"
when
	Item DenonPower changed from OFF to ON
then
/* 	var Number volume = (DenonVolumedB.state as DecimalType)+80 */
	sendCommand(DenonVolume, 40)
	logInfo("Rules", "DENON Volume initialized");
end

rule "DENON Volume to DB"
when
	Item DenonVolume changed
then
	if (DenonVolume.state == Undefined || DenonVolume.state == Uninitialized) 
		return false
	else {
		var Number volumedB = (DenonVolume.state as DecimalType)-80
		postUpdate(DenonVolumedB, volumedB)
	}
end
```