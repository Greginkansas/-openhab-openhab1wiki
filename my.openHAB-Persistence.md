my.openHAB Persistence
* [What is my.openHAB?](#what-is-myopenhab)
* [Installation and registration](#installation-and-registration)
* [Selective persistence](#selective-persistence)
* [Selectively adding specific items to my.openHAB](#selectively-adding-specific-items-to-myopenhab)
* [IFTTT and other cloud based services](#ifttt-and-other-cloud-based-services)
* [REST through my.openHAB](#rest-through-myopenhab)


# What is my.openHAB?

[my.openHAB.org](http://my.openhab.org) is a cloud service provided to you by openHAB and is well worth having a look at. After installing and registering, you can send Items and Events to the cloud where you can view them on the site.
Whilst that is not the most exciting thing in the world, it does then allow you to have access to your openHAB installation outside of your network via the iOS and Android apps without having to open up your home network.

It also allows you to connect to IFTTT and all the hundreds of channels they offer. Including IFTTT's own very handy [Maker Channel](https://ifttt.com/maker). Which essentially gives you a REST API through IFTTT into OpenHAB. Again without having to set up the network and port forwarding yourself.

# Installation and registration

First signup at [my.openhab.org](http://my.openhab.org) and follow the docs to install the binding, register and configure your setup.

### Configuring the .persist file

Create a file called `myopenhab.persist` in your persistence folder, and define the policy to store your data.

The policy on the my.openHAB site

The example on the my.openHAB site (for all items):

    Strategies {
        default = everyChange
    }
    Items {
        * : strategy = everyChange
    }

# Selective persistence

**Be aware that above example will persist ALL your items, and every change to my.openHAB cloud.**

There is a method to remove all data and from the my.openHAB cloud but no way to do it selectively. You can remove it all at once, or not at all. 

This means if you change the name of an item, the old "item" will still appear on my.openHAB cloud and also appear in your IFTTT dropdown. Also, if you set up my.openHAB with the demo configuration still setup, all of those items will also appear.

Any items such as a PIR or any type of sensor that sends constant updates to OpenHAB and will quickly clutter it up. So you probably don't need or want, the thousands of updates that will be logged to my.openHAB if you catchall.

People starting out should be aware of this due to the immense experimenting and testing that comes with learning OpenHAB. You will quickly find that the item list will contain a whole load of items that you no longer need.

## Selectively adding specific items to my.openHAB

Create a group to identify your items 

    Group    gMyOpenHAB

Then add items you know you need to log to that group. This will greatly help keep things under control.

e.g. Here only the on/off state of the switch is logged. There was no need to log every brightness state

    Switch Light_GF_Lounge_All    "Lounge All" (GF_Lounge, gMyOpenHAB) {milight="bridge01;6"}
    Dimmer Light_GF_Lounge_All_B  "Lounge All Brightness" (GF_Lounge) {milight="bridge01;6;brightness;27"}

Finally, configure your my.openHAB persistence file `myopenhab.persist` to only persist items in the group

````
Strategies {
    default = everyChange
}
Items {
    gMyOpenHAB* : strategy = everyChange
}
````

# IFTTT and other cloud based services

Be aware that **Sending persistence data to my.openHAB is required** if you use any data-driven cloud functions, [IFTTT](https://ifttt.com) integration for example. 

Meaning, if you want to control the brightness (or any item) using the OpenHAB IFTTT Channel, be sure that the item is in the gMyOpenHAB group, or it won't appear in the dropdown on IFTTT.

It is also required by the apps when used outside your local network.

[Official documentation](https://my.openhab.org/docs/persistence)

# REST through my.openHAB

You can also use REST through my.openHAB.org.

e.g.

To get the state of all items (remember my.openHAB will only display items that you have persisted to it)

    https://my.openhab.org/rest/items

You can specify:

    https://my.openhab.org/rest/items/Light_FF_Office_Desk

You can also send commands:

    https://my.openhab.org/CMD?Light_FF_Office_Desk=TOGGLE
    https://my.openhab.org/CMD?Light_FF_Office_Ceiling=ON

