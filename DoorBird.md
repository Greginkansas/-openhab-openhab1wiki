#DoorBird Integration#

The (limited) possibilities the [API](http://doorbird.com/api) supports can be used based on openHAB standard functionality:

##Live Video##
Integration directly in sitemap:

    Video url="http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/video.cgi" encoding="mjpeg"

##Open Door##
Item definition using HTTP binding:

    Switch DoorBird_DoorOpener "DoorBird Door Opener" (DoorBird)  { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/open-door.cgi]"

##Light On##
Item definition using HTTP binding:

    Switch DoorBird_DoorOpener "DoorBird Door Opener" (DoorBird)  { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/light-on.cgi]"

##History Images##
Integration directly in sitemap (20x):

    Image url="http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/history.cgi?index=1

##Notifications##
Item definition using HTTP binding:

    Switch DoorBird_MotionSensor_Register "DoorBird - Register Motion Sensor" (DoorBird) { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/CMD%3FDoorBird_MotionSensor%3DON&user=<openhab-user>&password=<openhab-password>&event=motionsensor&subscribe=1] >[OFF:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/CMD%3FDoorBird_MotionSensor%3DON&user=<openhab-user>&password=<openhab-password>&event=motionsensor&subscribe=0]" }
    Switch DoorBird_DoorBell_Register "DoorBird - Register Door Bell" (DoorBird) { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/CMD%3FDoorBird_DoorBell%3DON&user=<openhab-user>&password=<openhab-password>&event=doorbell&subscribe=1] >[OFF:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/CMD%3FDoorBird_DoorBell%3DON&user=<openhab-user>&password=<openhab-password>&event=doorbell&subscribe=0]" }

This then also requires additional items to receive the notifications:

    Switch DoorBird_MotionSensor "DoorBird Motion Sensor Triggered" (DoorBird)
    Switch DoorBird_DoorBell "DoorBird Door Bell Triggered" (DoorBird)

_Note: providing a user and password for the callback is not mandatory in case basic HTTP authentication is not enabled in openHAB._