##Background
Many zwave devices communicate of a basic radio protocol which can be intercepted or spoofed.  But ZWave also supports encrypted communications via the Security Command Class which is used for high value use cases such as door locks.  The Security Class provides extra protection to help prevent messages from being intercepted and/or spoofed.

##Warnings
- If you are using this code with a door lock, note that the toggle switch on the GUI may not represent the true state of the lock.  Please manually check that your doors are locked at night 
- The Security command classes in openhab are beta, use at your own risk!
- As with all modern crypto, the encryption is only as strong as the key.  Please take some time to generate a random key and do not use all zeros, etc

##Status
**General notes**
- Door locks have very limited range.  This is not specific to openhab and appears to be the case in general.  If the door lock is more than a few feet away from the controller, you will likely need a repeater (appliance module) for reliable communication
- The secure protocol has more overhead than normal communications.  For example, turning on a light is near instantaneous - you hit the switch in the UI and light comes on very quickly.  Secure messages take much longer as there are messages to exchange prior to sending the actual message.  When locking/unlocking a door, it's typical to take 5-10 seconds for the lock to respond.  It will then take additional time for the status to update on the UI

**What it does**
- Lock and unlock a door lock
- Reports battery percentage
- Update Lock status
- Set user codes - see [instructions](https://github.com/openhab/openhab/wiki/ZWave-Security-Testing/_edit#user-codes)

**Limitations/Known Issues**  not a wishlist, just things that are necessary for the bare minimum functionality that aren't working yet
- Do not do a habmin reinitialize on a door lock.  It has been known to cause issues and require a new secure repairing to restore functionality


##Instructions
**Get the Beta code**

1. Create a local clone of the dbadia openHAB repository by running "git clone https://github.com/dbadia/openhab" in a suitable folder.
1. Change to the beta testing branch by running "git checkout -b security-beta-test origin/security-beta-test"
1. Continue with step 2 at https://github.com/openhab/openhab/wiki/IDE-Setup 


**Setup**

1. enabled debug per https://github.com/openhab/openhab/wiki/Z-Wave-Binding#logging
1. edit openhab.cfg and add a new entry in the zwave section YOU MUST REPLACE each ## with random hex digits:  
`zwave:networkKey=0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##, 0x##`
1. If you have already paired the device with openhab you must unpair it using the exclusion function.  After it has been excluded, Stop openhab and delete the etc/zwave/node#.xml that corresponded to the device
1. hard reset the device.  NOTE: if this is a door lock, this will likely erase all door codes you have programmed!


**Steps to test**

1. Run the code downloaded above.  Note that pairing process is different than with other zwave devices. The zwave controller needs to stay connected to the machine running openhab and it needs to be very close to the secure device (a foot or less) during pairing.  
1. put the zwave controller into inclusion mode using habmin http://community-openhab-org.s3-eu-central-1.amazonaws.com/original/1X/63ec67c2a6dff6a42c8ef18e46333b0404953cb7.png
- trigger secure pair from the device per the instructions.  You should see lots of activity in the log file at this time.  The device make show a confirmation or make noise that it is complete, but that is often premature. After a minute or 2, check your logs for "Secure Inclusion complete" or "Secure Inclusion FAILED".  If it was complete, WAIT until you see "Node advancer: loop - DONE" before continuing to the next step.  This is the critical indication that openhab is done interrogating the device to see which commands it supports.  This literally took 4 minutes with my Yale lock, so be patient.  
1. If secure inclusion failed, you can stop the server right away.  Post your results to the thread with the device you are using and the full openhab.log file.  Before trying to secure pair again, you must exclude the device using habmin
1. Stop the openhab server.  You will now add the device to the items config file.  For example, door lock would require 3 new entries: 1) control the lock, 2) show the current state (requesting a lock/unlock doesn't mean it worked, and someone can manually lock/unlock at any time, so it's critical to NOT rely on the state of the switch), and 3) show battery status. For example: 
`Switch Door_Lock "Front door lock control" <none> (GF_Office) {zwave="#:command=door_lock"}`
`Contact Door_Basic "Front door lock status [%s]" <lock> (GF_Office) {zwave="#:command=door_lock,refresh_interval=120"}`
`Number Door_Corridor_Battery "Front door lock battery level [%d %%]" (GF_Office) { zwave="#:command=battery,refresh_interval=43200" }`
Be sure to replace # above with the id of your door lock from the secure pairing session
1. start openhab, wait for everything to initialize and check the logs for errors
1. Try the switch and wait 10 seconds.  If you hear the lock turn, great!  If not, the switch may have been in the wrong position to begin with, so try it again and wait 10 seconds

##User Codes

**Overview**

The user code command class allows the door entry codes to be set on the door lock by OH.  Currently this is a manual process which requires editing the node xml file.  Hopefully this will change in the future with OpenHab2.

The friendlyName value below is just a way for humans to track the codes.  The friendlyName is never sent to the door lock.  OH and the door lock track codes by the position in the userCodeList list.


**Adding codes**

1. Make a backup of all files in openhabhome\etc\zwave.  Move the backup outside of the zwave directory.
1. Open the node xml file for the door lock with a text editor.  Search for "<commandClass>USER_CODE</commandClass>" and you should see something like the example below.
**NOTE:** If you don't see the USER_CODE section, you probably paired the lock with OH before user code support was added to the security branch.  Do a git pull, then exclude and re-perform the secure inclusion process as described above
    ```
     <entry>
       <commandClass>USER_CODE</commandClass>
       <userCodeCommandClass>
         <version>1</version>
         <instances>1</instances>
         <numberOfUsersSupported>30</numberOfUsersSupported>
         <userCodeList/>
       </userCodeCommandClass>
     </entry>
    ```

1. Note the value of numberOfUsersSupported.  This is the maximum number of user codes you can store in the lock.  If this value is zero, than no codes can be stored
1. You will now edit the "<userCodeList/>" line.  Each code is given a friendly name to go along with the code.  For example to add the user "Dave" with the code "1234" it would look like this:

    ```
    <entry>
      <commandClass>USER_CODE</commandClass>
      <userCodeCommandClass>
        <version>1</version>
        <instances>1</instances>
        <numberOfUsersSupported>30</numberOfUsersSupported>
        <userCodeList>
          <userCode>
            <friendlyName>Dave</friendlyName>
            <code>1234</code>
          </userCode>
        </userCodeList>
      </userCodeCommandClass>
    </entry>
    ```

5. To add multiple codes, simply repeat the userCode block, like so:

    ```
    <entry>
      <commandClass>USER_CODE</commandClass>
      <userCodeCommandClass>
        <version>1</version>
        <instances>1</instances>
        <numberOfUsersSupported>30</numberOfUsersSupported>
        <userCodeList>
          <userCode>
            <friendlyName>Dave</friendlyName>
            <code>1234</code>
          </userCode>
          <userCode>
            <friendlyName>Bob</friendlyName>
            <code>5678</code>
          </userCode>
        </userCodeList>
      </userCodeCommandClass>
    </entry>
    ```

1. Once your edits are complete, restart OH and your changes should take effect shortly.

**Editing codes**

1. You can add or change codes at any time, just edit the node xml and restart OH
1. If you are removing an item from the list, set the code to 0 (zero) and recycle.  This will trigger OH to tell the door lock to remove the code in that position from the door lock.  Test it out to make sure the code no longer works, then you can remove that userCode block
