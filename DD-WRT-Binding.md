_**Note:** This Binding is available in 1.9.0 and later releases._

Documentation of the DD-WRT binding bundle

## Introduction

For installation of the binding, please see Wiki page [[Bindings]], or you can add [this JAR] (https://openhab.ci.cloudbees.com/job/openHAB1-Addons/lastSuccessfulBuild/artifact/bundles/binding/org.openhab.binding.ddwrt/target/org.openhab.binding.ddwrt-1.10.0-SNAPSHOT.jar) to your `addons` folder.

Adapt your openhab.cfg to your configuration:
* IP address of DD-WRT to connect to<BR>
    ddwrt:ip=192.168.1.1<BR>
    ddwrt:port=23<BR>

* You need to configure the user and password of your DD-WRT<BR>
    ddwrt:username=root<BR>
    ddwrt:password=xxxxxxx<BR>

* Interface for the 2.4 GHz wifi<BR>
    ddwrt:interface_24=ath0<BR>
* Interface for the 5 GHz wifi<BR>
    ddwrt:interface_50=ath1<BR>
* Virtuall-Interface for the guest wifi<BR>
    ddwrt:interface_guest=ath0.1<BR>


## Prepare your DD-WRT device
* You have to activate the telnet connection in the DD-WRT web interface.
* The changing of the telnet port in the DD-WRT web interface is not always working. Test it with a telnet command shell.

## Generic Item Binding Configuration

In order to bind an item to the DD-WRT device, you need to provide configuration settings. The easiest way to do so is to add some binding information in your item file (in the folder `configurations/items`). 

## Switching WIFI

The following items switch WIFI, GUEST_WIFI, and the NAME of the device as string:

    String DEVICE_NAME {ddwrt="routertype"}
    Switch WIFI_24     {ddwrt="wlan24"}
    Switch WIFI_50     {ddwrt="wlan50"}
    Switch WIFI_GUEST  {ddwrt="wlanguest"}

The guest network is usually a virtual network device. There is a bug in the DD-WRT firmware. The activation of this interface needs a workaround so it takes some seconds more as the native devices.

Tested with Archer V2 and DD-WRT v3.0-r30880 std (11/14/16)

## Introduction - openHAB v2.x
The current version of this binding is not yet converted to openHAB 2.x formatting but it does work if you follow the instructions below.  For those that want to help make it 2.x compliant, please feel free to edit these instructions.

### Step 1 - Prepare Router
* Activate the telnet interface via the web interface (Services -> Services Telnet Section -> Enable).
* Make note of the port number (Services -> Services Secure Shell -> Port).
* Connect to the router's command line using Telnet (using your favorite telnet client).
* Get a list of active interfaces using the `ifconfig` command.
* Make note of the different interface names. (ath0, br0, eth2, wl0.1 etc).

### Step 2 - Install the dd-WRT binding JAR files
* Download the current [binding JAR] (https://openhab.ci.cloudbees.com/job/openHAB1-Addons/lastSuccessfulBuild/artifact/bundles/binding/org.openhab.binding.ddwrt/target/org.openhab.binding.ddwrt-1.10.0-SNAPSHOT.jar) file from openhab cloudbees.
* Save the file to the /sys/addons folder

### Step 3 - Create your ddwrt.cfg file
* In your preferred text editor copy and past the following:

`############################### DD-WRT Binding ####################################`  
`#`  
`#`  
`# IP address of DD-WRT to connect to.  Replace XXX.XXX.XXX.XXX with your routers Local IP address.`  
`# Please note that the router and openHAB server should be on the same network.`  
`#`  
`ddwrt:ip=XXX.XXX.XXX.XXX`  
`#`  
`# Port Number for the Telnet connection.  Replace YYYYY with the port number.` 
`#`  
`ddwrt:port=YYYYY`  
`#`  
`# Connection information for to access the routers command line.`  
`# Only needed if Password Login is activated on the router.`  
`# Replace USERNAME and USERPASSWORD fileds below with your connecton details.`  
`#`  
`ddwrt:username=USERNAME`  
`ddwrt:password=USERPASSWORD`  
`#`  
`# Interface 1. Normally assigned to the 2.4 GHz wifi physical interface (ath0 or wl0).`  
`# Replace INTERFACE1 with the name of the interface you want to trigger.`  
`#`  
`ddwrt:interface_24=INTERFACE1`  
`#`  
`# Interface 2. Normally for the 5 GHz wifi physical interface (ath1 or wl1)`  
`# Replace INTERFACE2 with the name of the interface you want to trigger.`  
`#`  
`ddwrt:interface_50=INTERFACE2`  
`#`  
`# Interface 3. Normally used for Virtual-Interface such as guest wifi (ath1.1 or wl0.1).`  
`# This interface switch triggers additional reset commands on the router.`  
`# Replace INTERFACE3 with the name of the interface you want to trigger.`  
`#`  
`ddwrt:interface_guest=INTERFACE3`  
`#`  
`#`  
`#####################################################################################`

* Save the file as ddwrt.cfg to the /conf/Services folder.

### Step 4 - Add dd-WRT Items to openHAB
* In your preferred text editor copy and past the following:

`// DD-WRT Binding //`  
`String  DEVICE_NAME  "NAME OF ROUTER"  <signal>  (GROUP1, GROUP2)  {ddwrt="routertype"}`  
`Switch  INTERFACE1_NAME  "INTERFACE1 NAME" <signal>  (GROUP1, GROUP2)  {ddwrt="wlan24"}`  
`Switch  INTERFACE2_NAME  "INTERFACE2 NAME" <signal>  (GROUP1, GROUP2) 	{ddwrt="wlan50"}`  
`Switch  INTERFACE3_NAME  "INTERFACE3 NAME" <signal>  (GROUP1, GROUP2)  {ddwrt="wlanguest"}`  

#### Customize the Item Names
* Replace DEVICE_NAME, INTERFACE1_NAME, INTERFACE2_NAME and INTERFACE3_NAME 
* Ensure they are unique througout your openHAB instalation

#### Customize the Item Descriptions
* Replace "NAME OF ROUTER", "INTERFACE1 NAME", "INTERFACE2 NAME" and "INTERFACE3 NAME"

#### Customize the Item Icons
* Replace or Delete the "<signal>" fields

#### Customize the Item Group Membership
* Replace or Delete the "(GROUP1, GROUP2)" fields

#### Save the file to openHAB
* Save the file as ddwrt.cfg in the /conf/Items folder

### Step 5 - Follow-up
* Please remember to backup your router before activating and using this binding.  Turning off the wrong interface may result in the loss of network access, therfore preventing openHAB from turining them back on.
* Your routers chipset and configuration will determin the interfaces you can control.  Any of the interfaces listed when the ifcomand was run can be attached to the binding.  


