# DoorBird Integration

The (limited) possibilities the [API](http://doorbird.com/api) supports can be used based on openHAB standard functionality:

## Live Video
Integration directly in sitemap:

    Video url="http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/video.cgi" encoding="mjpeg"

## Live Image
The integration could be done similarly to Live Video, i.e. directly in the sitemap:

    Image url="http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/image.cgi"

However, this feature is more useful if used in combination with a rule - e.g. to save an image every time the motion sensor was triggered (this essentially creates a surveillance camera):

    rule "Save Image when Motion Detected"
        when
            Item DoorBird_MotionSensor received command ON
        then
            var t = now
            var cmd = 'wget http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/image.cgi -O /var/doorbird/'
            cmd = cmd + t.getYear
            if (t.getMonthOfYear < 10) cmd = cmd + '0'
            cmd = cmd + t.getMonthOfYear
            if (t.getDayOfMonth < 10) cmd = cmd + '0'
            cmd = cmd + t.getDayOfMonth
            if (t.getHourOfDay < 10) cmd = cmd + '0'
            cmd = cmd + t.getHourOfDay
            if (t.getMinuteOfHour < 10) cmd = cmd + '0'
            cmd = cmd + t.getMinuteOfHour
            if (t.getSecondOfMinute < 10) cmd = cmd + '0'
            cmd = cmd + t.getSecondOfMinute
            cmd = cmd + '.jpg'
            
            executeCommandLine(cmd)
            
            sendCommand(DoorBird_MotionSensor, OFF)
    end

The above script simply creates a filename in the format YYYYMMDDHHMMSS.jpg and puts it into /var/doorbird. Ensure that directory permissions are set accordingly.

## Open Door
Item definition using HTTP binding:

    Switch DoorBird_DoorOpener "DoorBird Door Opener" (DoorBird)  { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/open-door.cgi]"

## Light On
Item definition using HTTP binding:

    Switch DoorBird_DoorOpener "DoorBird Door Opener" (DoorBird)  { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/light-on.cgi]"

## History Images
Integration directly in sitemap (20x):

    Image url="http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/history.cgi?index=1

## Notifications
Item definition using HTTP binding OH1.x:

    Switch DoorBird_MotionSensor_Register "DoorBird - Register Motion Sensor" (DoorBird) { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/CMD%%3FDoorBird_MotionSensor%%3DON&user=<openhab-user>&password=<openhab-password>&event=motionsensor&subscribe=1] >[OFF:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/CMD%%3FDoorBird_MotionSensor%%3DON&user=<openhab-user>&password=<openhab-password>&event=motionsensor&subscribe=0]" }
    Switch DoorBird_DoorBell_Register "DoorBird - Register Door Bell" (DoorBird) { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/CMD%%3FDoorBird_DoorBell%%3DON&user=<openhab-user>&password=<openhab-password>&event=doorbell&subscribe=1] >[OFF:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/CMD%%3FDoorBird_DoorBell%%3DON&user=<openhab-user>&password=<openhab-password>&event=doorbell&subscribe=0]" }
    Switch DoorBird_DoorOpen_Register "DoorBird - Register Door Open" (DoorBird) { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/CMD%%3FDoorBird_DoorOpen%%3DON&user=<openhab-user>&password=<openhab-password>&event=dooropen&subscribe=1] >[OFF:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/CMD%%3FDoorBird_DoorOpen%%3DON&user=<openhab-user>&password=<openhab-password>&event=dooropen&subscribe=0]" }

For OH2, additional characters need escaped (the colon) in the callback URL using the HTTP binding, and the CMD URL path is now classicUI/CMD (and requires the Classic UI to be installed):

    Switch DoorBird_MotionSensor_Register "DoorBird - Register Motion Sensor" (DoorBird) { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http%%3A//<openhab-ip>%%3A<openhab-port>/classicui/CMD%%3FDoorBird_MotionSensor%%3DON&user=<openhab-user>&password=<openhab-password>&event=motionsensor&subscribe=1] >[OFF:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http%%3A//<openhab-ip>%%3A<openhab-port>/classicui/CMD%%3FDoorBird_MotionSensor%%3DON&user=<openhab-user>&password=<openhab-password>&event=motionsensor&subscribe=0]" }
    Switch DoorBird_DoorBell_Register "DoorBird - Register Door Bell" (DoorBird) { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http%%3A//<openhab-ip>%%3A<openhab-port>/classicui/CMD%%3FDoorBird_DoorBell%%3DON&user=<openhab-user>&password=<openhab-password>&event=doorbell&subscribe=1] >[OFF:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http%%3A//<openhab-ip>%%3A<openhab-port>/classicui/CMD%%3FDoorBird_DoorBell%%3DON&user=<openhab-user>&password=<openhab-password>&event=doorbell&subscribe=0]" }
    Switch DoorBird_DoorOpen_Register "DoorBird - Register Door Open" (DoorBird) { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http%%3A//<openhab-ip>%%3A<openhab-port>/classicui/CMD%%3FDoorBird_DoorOpen%%3DON&user=<openhab-user>&password=<openhab-password>&event=dooropen&subscribe=1] >[OFF:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http%%3A//<openhab-ip>%%3A<openhab-port>/classicui/CMD%%3FDoorBird_DoorOpen%%3DON&user=<openhab-user>&password=<openhab-password>&event=dooropen&subscribe=0]" }

This then also requires additional items to receive the notifications:

    Switch DoorBird_MotionSensor "DoorBird Motion Sensor Triggered" (DoorBird)
    Switch DoorBird_DoorBell "DoorBird Door Bell Triggered" (DoorBird)
    Switch DoorBird_DoorOpen "DoorBird Door Opener Triggered" (DoorBird)

_Note: providing a user and password for the callback is not mandatory in case basic HTTP authentication is not enabled in openHAB.