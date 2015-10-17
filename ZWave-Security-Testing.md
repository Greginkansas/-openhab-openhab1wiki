##Background
Many zwave devices communicate of a basic radio protocol which can be intercepted or spoofed.  But ZWave also supports encrypted communications via the Security Command Class which is used for high value use cases such as door locks.  The Security Class provides extra protection to help prevent messages from being intercepted and/or spoofed.

##Warnings
- If you are using this code with a door lock, note that the toggle switch on the GUI may not represent the true state of the lock.  Please manually check that your doors are locked at night 
- The Security command classes in openhab are beta, use at your own risk!
- As with all modern crypto, the encryption is only as strong as the key.  Please take some time to generate a random key and do not use all zeros, etc

##Status
**What it does**
- Lock and unlock a door lock
- Reports battery percentage

**What it doesn't do (yet!)**  not a wishlist, just things that are necessary for the bare minimum functionality that aren't working yet
- Update Lock status (when someone manually opens/closes)
- Trying to perform lock/unlock commands too quickly will result in failure.  Ideally some sort of message would appear preventing the user from doing this

##Instructions
**Get the Beta code**

1. Create a local clone of the dbadia openHAB repository by running "git clone https://github.com/dbadia/openhab" in a suitable folder.
1. Change to the 1.6 branch by running "git checkout -b security-beta-test origin/security-beta-test"
1. Continue with step 2 at https://github.com/openhab/openhab/wiki/IDE-Setup 


**Setup**

1. enabled debug per https://github.com/openhab/openhab/wiki/Z-Wave-Binding#logging
1. edit openhab.cfg and add a new entry in the zwave section YOU MUST REPLACE each ## with random hex digits:  
`zwave:networkKey=0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##`
1. If you have already paired the device with openhab you must unpair it using the exclusion function.  After it has been excluded, Stop openhab and delete the etc/zwave/node#.xml that corresponded to the device
1. hard reset the device.  NOTE: if this is a door lock, this will likely erase all door codes you have programmed!


**Steps to test**

1. Run the code downloaded above.  Note that pairing process is different than with other zwave devices. The zwave controller needs to stay connected to the machine running openhab and it needs to be very close to the secure device (a foot or less).
1. put the zwave controller into inclusion mode using habmin http://community-openhab-org.s3-eu-central-1.amazonaws.com/original/1X/63ec67c2a6dff6a42c8ef18e46333b0404953cb7.png
- trigger secure pair from the device per the instructions.  You should see lots of activity in the log file at this time.  After a minute or 2, check your logs for "Secure Inclusion complete" or "Secure Inclusion FAILED"
1. If secure inclusion failed, post your results to this thread with the device you are using and the full openhab.log file.  If secure inclusoin worked, continue to the next step.
1. Stop the openhab server.  You will now add the device to the items config file.  For example, door lock would require 3 new entries: 1) control the lock, 2) show the current state (requesting a lock/unlock doesn't mean it worked, and someone can manually lock/unlock at any time, so it's critical to NOT rely on the state of the switch), and 3) show battery status. For example: 
`Switch Door_Lock "Front door lock control" (GF_Office) {zwave="#:command=door_lock"}`
`Contact Door_Basic "Front door sensor" (GF_Office) {zwave="#:command=sensor_binary,refresh_interval=10"} `
`Number Door_Corridor_Battery "Front door  battery level [%d %%]" (GF_Office) { zwave="#:command=battery" }`
Be sure to replace # above with the id of your door lock from the secure pairing session
1. start openhab, wait for everything to initialize and check the logs for errors
1. Try the switch and wait 10 seconds.  If you hear the lock turn, great!  If not, the switch may have been in the wrong position to begin with, so try it again and wait 10 seconds