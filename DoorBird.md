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
The outbound part is rather straightforward and can be implemented with an item definition using HTTP binding:

    Switch DoorBird_MotionSensor_Register "DoorBird - Register Motion Sensor" (DoorBird) { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/doorbird-callback/motion-sensor&user=<openhab-user>&password=<openhab-password>&event=motionsensor&subscribe=1] >[OFF:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/doorbird-callback/motion-sensor&user=<openhab-user>&password=<openhab-password>&event=motionsensor&subscribe=0]" }
    Switch DoorBird_DoorBell_Register "DoorBird - Register Door Bell" (DoorBird) { http=">[ON:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/doorbird-callback/door-bell&user=<openhab-user>&password=<openhab-password>&event=doorbell&subscribe=1] >[OFF:GET:http://<doorbird-user>:<doorbird-password>@<doorbird-ip>/bha-api/notification.cgi?url=http://<openhab-ip>:<openhab-port>/doorbird-callback/door-bell&user=<openhab-user>&password=<openhab-password>&event=doorbell&subscribe=0]" }

DoorBird can only send a GET request to a web ressource you have to provide in the registration.
This can be solved using a "gateway" based on a servlet that accepts the GET request from the DoorBird and notifies the corresponding item in openHAB via REST. The implementation and according documentation can be found here: https://github.com/bern77/DoorBird-Callback

This then also requires additional items to receive the notifications:

    Switch DoorBird_MotionSensor "DoorBird Motion Sensor Triggered" (DoorBird)
    Switch DoorBird_DoorBell "DoorBird Door Bell Triggered" (DoorBird)