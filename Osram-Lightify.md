# Osram Lightify Integration - Updated for openHAB 2

Usually Osram Lightify bulbs work nicely when connected to a Hue bridge which allows using the corresponding binding.
However, if for some reason somebody is as unfortunate as me, i.e. the mix of bulbs in the house is located in a way that prevents that the (apparently proprietary) range extension functionality between devices doesn't ensure the bulbs can be reached (directly or indirectly) by the bridge, this approach might be useful.

Osram has published an API which could be used to control the bulbs. However, as this is handled via an Osram server, both privacy as well as latency issues render this a rather useless solution.

Instead, I came across a Python implementation using the binary protocol of the Lightify Gateway, which works very fast.
My solution is based on the following components / code:

## Python Script

### Installation/Preparation
    pip install lightify

Run a simple Python script (changing the IP address to that of the Lightify Gateway)
    from lightify import Lightify
    
    lightify = Lightify("<IP ADDRESS>")
    lightify.update_all_light_status()
    lights = lightify.lights()
    print lights

You then get a list of indexes in a format 'XXXXXXXXXXXXXXXXXXXL' (X being a digit). You can use

    lightify.lights()[XXXXXXXXXXXXXXXXXXXL].set_onoff(1)

to identify which index corresponds to which bulb.

### mylightify.py
    from lightify import Lightify
    import argparse
    
    # Light index
    Room1 = XXXXXXXXXXXXXXXXXXXL
    Room2 = XXXXXXXXXXXXXXXXXXXL
    #...

    # process command line parameters
    parser = argparse.ArgumentParser()
    parser.add_argument('--bridge', help='IP Address of Lightify bridge')
    parser.add_argument('--device', help='ID of the Lightify device')
    parser.add_argument('--onoff', help='indicates if device should be switched into ON or OFF state', type=int)
    parser.add_argument('--brightness', help='level of brightness to be used', type=float)
    parser.add_argument('--saturation', help='saturation to be used', type=float)
    parser.add_argument('--red', help='level of red to be used', type=float)
    parser.add_argument('--green', help='level of green to be used', type=float)
    parser.add_argument('--blue', help='level of blue to be used', type=float)
    parser.add_argument('--temperature', help='colour temperature to be used', type=int)
    parser.add_argument('--time', help='time for transition', type=int)
    args = parser.parse_args()
    
    # connect to Lightify bridge
    if args.bridge != None:
        lightify = Lightify(args.bridge)
        lightify.update_all_light_status()
        lights = lightify.lights()
    
        if args.device != None:
    
            # device selection
            l = None
            if args.device == 'Room1':
                l = Room1
            elif args.device == 'Room2':
                l = Room2
            #...
            
            if l != None:
                light = lightify.lights()[l]
    
                # define RGB
                red = None
                green = None
                blue = None
                if args.red != None:
                    red = int(round(args.red))
                if args.green != None:
                    green = int(round(args.green))
                if args.blue != None:
                    blue = int(round(args.blue))
                
                time = 0
                if args.time != None:
                    time = args.time
    
                if args.onoff != None and args.onoff == 0:
                    light.set_onoff(0)
                else:
                    light.set_onoff(1)
                    if red != None and green != None and blue != None:
                        light.set_rgb(red, green, blue, time)
                    if args.brightness != None:
                        light.set_luminance(args.brightness, time)
                    if args.temperature != None:
                        light.set_temperature(args.temperature, time)

I placed the script into */openhab/scripts*.

## openHAB Integration

### Items
Assuming there are the following item definition:

    Switch Lightify1_OnOff
    Color  Lightify1_Colour
    Dimmer Lightify1_Dimmer

### Rules
Assuming there is a rule doing something like this:

    var DecimalType hue = new DecimalType(240)
    var PercentType sat = new PercentType(100)
    var PercentType bright = new PercentType(100)
    var HSBType light = new HSBType(hue, sat, bright)
    sendCommand(Lightify1_Colour, light)

In order to make the Lightify bulb behave like there was a binding I've added an additional Rules file:

### lightify.rules
    import org.eclipse.xtext.xbase.lib.Procedures$Procedure1
    import org.eclipse.xtext.xbase.lib.Procedures$Procedure3

    val String LIGHTIFY_SCRIPT = "python /openhab/scripts/mylightify.py --bridge <IP ADDRESS> "
    
    val Procedures$Procedure3<SwitchItem, DimmerItem, String> dimLightifyDevice = [
        SwitchItem s,
        DimmerItem d,
        String device |
    
        var cmd = LIGHTIFY_SCRIPT + '--device ' + device
        if (s.state == OFF) {
            cmd = cmd + ' --onoff 0'
        } else {
            if (d.state != NULL) {
                var b = (d.state as DecimalType).doubleValue
                cmd = cmd + ' --onoff 1 --brightness ' + b
            } else {
                cmd = cmd + ' --onoff 1'
            }
        }
        executeCommandLine(cmd)
    ]
    
    val Procedures$Procedure1<String> switchOffLightifyDevice = [
        String device |
    
            executeCommandLine(LIGHTIFY_SCRIPT + '--device ' + device + ' --onoff 0')
    ]
    
    val Procedures$Procedure3<SwitchItem, ColorItem, String> updateLightifyDevice = [ 
        SwitchItem s,
        ColorItem c,
        String device |
    
            var cmd = LIGHTIFY_SCRIPT + '--device ' + device
            if (s.state == OFF) {
                cmd = cmd + ' --onoff 0'
            } else {
                if (c.state != NULL) {
                    var light = c.state as HSBType
                    var red = light.red / 100 * 255
                    var green = light.green / 100 * 255
                    var blue = light.blue / 100 * 255
                    cmd = cmd + ' --onoff 1 --brightness ' + light.brightness + ' --red ' + red + ' --green ' + green + ' --blue ' + blue
                } else {
                    cmd = cmd + ' --onoff 1'
                }
            }
            executeCommandLine(cmd)
    ]

Note: You have to set the IP Address for your Lightify Gateway accordingly.
You can then easily add rules for the individual bulbs:

    rule "Lightify Workaround Lightify1"
        when
            Item Lightify1_OnOff changed or
            Item Lightify1_Colour changed
        then
            updateLightifyDevice.apply(Lightify1_OnOff, Lightify1_Colour, 'Room1')
    end
