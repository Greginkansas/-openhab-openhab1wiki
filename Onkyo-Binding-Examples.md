Here are some examples of configurations for various aspects of Onkyo receivers:

* [NET/USB](Onkyo-Binding-Examples#NetUsb)

***

#NetUsb

##Navigation

###Menu List Display

When using openHAB, it may be desirable to be able to control your A/V receiver without being able to see the video output of the receiver. This may be the case when you are using extra zone outputs for whole house audio, etc.. Onkyo receivers implement a hierarchical menu structure where each page of menu can contain up to ten items. Older generation (circa 2010) and newer generation (circa 2014) were more Model View Control (MVC) friendly. Older units send menu pages one item at a time, but they send all ten items regardless of whether or not they are all used. Newer units will send the complete menu page in one status update. Units made in between are some what problematic as they send menu pages one item at a time, but they only send as many items as are on the page. This causes problems when one page has say ten items, and the next only has eight. There is no easy way to know that the last two menu items should be cleared as no end of menu indication is given.

My first attempt at solving this problems was to clear all menu items when the first item was received. This didn't work as expected as openHAB executes rules in different threads and there is no guarantee that they will execute in the order they were invoked. This would result in random menu items missing as the rule for the first item executed after they had been updated. I wasn't able to find a good solution for this, but I was able to get things functional by delaying the execution of all rules except the one for the first item. Older receivers don't need this and don't need to clear menu items as all items are sent on menu page changes.

Onkyo receivers transmit menu items with a string matching the following REGEX:

NLSU[0-9]-[0-9a-zA-Z ]*

There may be other valid characters after the '-'. The 'U' in "NLSU" is for Unicode. Old receivers running old firmware images may have an 'A' here for ASCII.

Onkyo receivers transmit the current cursor position (within the menu page) with a string matching the following REGEX:

NLSC[0-9][CP]

The last character is a 'C' when the cursor moves within the page. The last character is a 'P' when a page change occurs. The NETUSB_OP_UP and NETUSB_OP_DOWN commands can be used to move the cursor up/down within the page. The NETUSB_OP_LEFT and NETUSB_OP_RIGHT commands can be used to scroll the current menu page up/down. The NETUSB_OP_RETURN command can be used to move up in the menu hierarchy. The NETUSB_OP_SELECT command can be used to select the current menu item. The "NLSL[0-9]" command can be used to randomly select a menu item (this is not supported on older models). These elements can be combined with rules and dynamic color to implement a reasonable user interface.

For menu list display a string item is used for each item in the menu (ten strings). These strings capture the raw status updates from the receiver, but are not displayed. There is a rule for each menu item that removes the overhead characters and posts the value to another string item for display. 

Dynamic color is used to indicate the current cursor position. In order to do this an item is used to capture the raw cursor position from the receiver. A rule is run when it changes to strip out the cursor line and post it to another item that is used to control the dynamic color (this simplifies the color rules and is useful later).
  
###Menu List Selection

I have included "NLSL[0-9]" commands on the displayed menu item strings in the hopes that openHAB will eventually support some type of selectable text that sends a command instead of going to a URL. Then the user can just select the menu item on receivers that support "NLSL[0-9]" commands. For older receivers, the NETUSB_OP_SELECT command is used. The user needs to navigate to the desired menu item before this command is sent. Setpoint elements are used for navigation as they provide a more compact arrangement.

##Items
```
//
// NET/USB
//
// Controls
Switch onkyoNETPlay         "Play"              (gOnkyo1)   {onkyo="ON:onkyo1:NETUSB_OP_PLAY", autoupdate="false"}
Switch onkyoNETPause        "Pause"             (gOnkyo1)   {onkyo="ON:onkyo1:NETUSB_OP_PAUSE", autoupdate="false"}
Switch onkyoNETStop         "Stop"              (gOnkyo1)   {onkyo="ON:onkyo1:NETUSB_OP_STOP", autoupdate="false"}
Switch onkyoNETTrackUp      "Track Up"          (gOnkyo1)   {onkyo="ON:onkyo1:NETUSB_OP_TRACKUP", autoupdate="false"}
Switch onkyoNETTrackDown    "Track Down"        (gOnkyo1)   {onkyo="ON:onkyo1:NETUSB_OP_TRACKDWN", autoupdate="false"}
Switch onkyoNETFF           "Fast Forward"      (gOnkyo1)   {onkyo="ON:onkyo1:NETUSB_OP_FF", autoupdate="false"}
Switch onkyoNETREW          "Rewind"            (gOnkyo1)   {onkyo="ON:onkyo1:NETUSB_OP_REW", autoupdate="false"}
Number onkyoNETService      "Service"           (gOnkyo1)   {onkyo="INIT:onkyo1:#NSVQST, *:onkyo1:#NSV%02X0"}
Number onkyoNETSelectList   "Select List Item"  (gOnkyo1)   {onkyo="*:onkyo1:#NLSL%01X"}
Switch onkyoNETUp           "Up"                (gOnkyo1)   {onkyo="ON:onkyo1:NETUSB_OP_UP", autoupdate="false"}
Switch onkyoNETDown         "Down"              (gOnkyo1)   {onkyo="ON:onkyo1:NETUSB_OP_DOWN", autoupdate="false"}
Switch onkyoNETLeft         "Left"              (gOnkyo1)   {onkyo="ON:onkyo1:NETUSB_OP_LEFT", autoupdate="false"}
Switch onkyoNETRight        "Right"             (gOnkyo1)   {onkyo="ON:onkyo1:NETUSB_OP_RIGHT", autoupdate="false"}
Switch onkyoNETReturn       "Return"            (gOnkyo1)   {onkyo="ON:onkyo1:NETUSB_OP_RETURN", autoupdate="false"}
Switch onkyoNETSelect       "Select"            (gOnkyo1)   {onkyo="ON:onkyo1:NETUSB_OP_SELECT", autoupdate="false"}
String onkyoNETCursor       "Cursor [%s]"       (gOnkyo1)   {onkyo="*:onkyo1:#NLSC"}
String onkyoNETList0        "1 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSU0"}
String onkyoNETList1        "2 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSU1"}
String onkyoNETList2        "3 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSU2"}
String onkyoNETList3        "4 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSU3"}
String onkyoNETList4        "5 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSU4"}
String onkyoNETList5        "6 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSU5"}
String onkyoNETList6        "7 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSU6"}
String onkyoNETList7        "8 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSU7"}
String onkyoNETList8        "9 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSU8"}
String onkyoNETList9        "10 [%s]"           (gOnkyo1)   {onkyo="*:onkyo1:#NLSU9"}
String onkyoNETSel0         "1 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSL0", autoupdate="false"}
String onkyoNETSel1         "2 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSL1", autoupdate="false"}
String onkyoNETSel2         "3 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSL2", autoupdate="false"}
String onkyoNETSel3         "4 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSL3", autoupdate="false"}
String onkyoNETSel4         "5 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSL4", autoupdate="false"}
String onkyoNETSel5         "6 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSL5", autoupdate="false"}
String onkyoNETSel6         "7 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSL6", autoupdate="false"}
String onkyoNETSel7         "8 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSL7", autoupdate="false"}
String onkyoNETSel8         "9 [%s]"            (gOnkyo1)   {onkyo="*:onkyo1:#NLSL8", autoupdate="false"}
String onkyoNETSel9         "10 [%s]"           (gOnkyo1)   {onkyo="*:onkyo1:#NLSL9", autoupdate="false"}
// Information
String onkyoNETArtist       "Artist [%s]"       (gOnkyo1)   {onkyo="INIT:onkyo1:NETUSB_SONG_ARTIST_QUERY"}
String onkyoNETAlbum        "Album [%s]"        (gOnkyo1)   {onkyo="INIT:onkyo1:NETUSB_SONG_ALBUM_QUERY"}
String onkyoNETTitle        "Title [%s]"        (gOnkyo1)   {onkyo="INIT:onkyo1:NETUSB_SONG_TITLE_QUERY"}
String onkyoNETTrack        "Track [%s]"        (gOnkyo1)   {onkyo="INIT:onkyo1:NETUSB_SONG_TRACK_QUERY"}
String onkyoNETTime         "Time [%s]"         (gOnkyo1)   {onkyo="INIT:onkyo1:NETUSB_SONG_ELAPSEDTIME_QUERY"}
String onkyoNETPlayStatus   "Play Status [%s]"  (gOnkyo1)   {onkyo="INIT:onkyo1:NETUSB_PLAY_STATUS_QUERY"}
// Proxy
Number onkyoNETPage         "Page"              (gOnkyo1)
Number onkyoNETCursorPos    "Cursor"            (gOnkyo1)
```

##Sitemap
```
            Text label="NET/USB" icon="sofa" {
                Frame label="Information" {
                    Text item=onkyoNETArtist
                    Text item=onkyoNETAlbum
                    Text item=onkyoNETTitle
                    Text item=onkyoNETTrack
                    Text item=onkyoNETTime
                }
                Frame label="Control"{
                    Text label="Navigation" icon="sofa" {
                        Frame {
                            Selection   item=onkyoNETService mappings=[0="Media Server (DLNA)", 1=Favorite, 2=vTuner, 3=SIRIUS, 6="Last.fm", 14="TuneIn Radio"]
//                          Selection   item=onkyoNETSelectList mappings=[0="1", 1="2", 2="3", 3="4", 4="5", 5="6", 6="7", 7="8", 8="9", 9="10"]
                            Switch      item=onkyoNETReturn
                            Switch      item=onkyoNETSelect
                            Setpoint    item=onkyoNETPage minValue=0 maxValue=2 step=1
                            Setpoint    item=onkyoNETCursorPos minValue=0 maxValue=9 step=1
                        }
                        Frame label="List" {
                            Text        item=onkyoNETSel0 valuecolor=[onkyo2NETCursorPos=="9"="fuchsia"]
                            Text        item=onkyoNETSel1 valuecolor=[onkyo2NETCursorPos=="8"="fuchsia"]
                            Text        item=onkyoNETSel2 valuecolor=[onkyo2NETCursorPos=="7"="fuchsia"]
                            Text        item=onkyoNETSel3 valuecolor=[onkyo2NETCursorPos=="6"="fuchsia"]
                            Text        item=onkyoNETSel4 valuecolor=[onkyo2NETCursorPos=="5"="fuchsia"]
                            Text        item=onkyoNETSel5 valuecolor=[onkyo2NETCursorPos=="4"="fuchsia"]
                            Text        item=onkyoNETSel6 valuecolor=[onkyo2NETCursorPos=="3"="fuchsia"]
                            Text        item=onkyoNETSel7 valuecolor=[onkyo2NETCursorPos=="2"="fuchsia"]
                            Text        item=onkyoNETSel8 valuecolor=[onkyo2NETCursorPos=="1"="fuchsia"]
                            Text        item=onkyoNETSel9 valuecolor=[onkyo2NETCursorPos=="0"="fuchsia"]
                        }
                    }
                    Text label="Transport" icon="sofa" {
                        Switch      item=onkyoNETPlay
                        Switch      item=onkyoNETPause
                        Switch      item=onkyoNETStop
                        Switch      item=onkyoNETTrackUp
                        Switch      item=onkyoNETTrackDown
                        Switch      item=onkyoNETFF
                        Switch      item=onkyoNETREW
                    }
                }
            }
```

##Rules
```java
import org.openhab.core.library.types.*
import org.openhab.core.library.items.*
import org.openhab.core.persistence.*
import org.openhab.model.script.actions.*

import java.util.concurrent.locks.ReentrantLock
import java.util.List
import java.util.ArrayList


var ReentrantLock                       onkyoLock  = new java.util.concurrent.locks.ReentrantLock()
var Integer                             onkyoCursorPos


rule "Init"
    when
        System started
    then
        onkyoLock.lock()
        try {
            onkyoCursorPos = 9
            postUpdate(onkyoNETCursorPos, onkyoCursorPos)
            postUpdate(onkyoNETPage, 1)
        } finally {
           onkyoLock.unlock()
        }
end


rule "Update Page"
    when
        Item onkyoNETPage changed
    then
        onkyoLock.lock()
        try {
            logDebug("onkyo.rules", onkyoNETPage.state.toString())
 
            if (onkyoNETPage.state > 1) {
                sendCommand(onkyoNETLeft, ON)               
            } else if (onkyoNETPage.state < 1) {
                sendCommand(onkyoNETRight, ON)              
            }
            postUpdate(onkyoNETPage, 1)
        } finally {
           onkyoLock.unlock()
        }
end


rule "Update CursorPos"
    when
        Item onkyoNETCursorPos changed
    then
        onkyoLock.lock()
        try {
            logDebug("onkyo.rules", onkyoNETCursorPos.state.toString())
 
            if (onkyoNETCursorPos.state < onkyoCursorPos) {
                sendCommand(onkyoNETDown, ON)               
            } else if (onkyoNETCursorPos.state > onkyoCursorPos) {
                sendCommand(onkyoNETUp, ON)             
            }
        } finally {
           onkyoLock.unlock()
        }
end


rule "Update Cursor"
    when
        Item onkyoNETCursor changed
    then
        onkyoLock.lock()
        try {
            logDebug("onkyo.rules", onkyoNETCursor.state.toString())
            onkyoCursorPos = 9 - new Integer(onkyoNETCursor.state.toString.substring(1,1))
            postUpdate(onkyoNETCursorPos, onkyoCursorPos)
        } finally {
           onkyoLock.unlock()
        }
end


/*
 * This rule processes Onkyo list updates and removes leading status type characters.
 */
rule "Update List 0 Item"
    when
        Item onkyoNETList0 received update
    then
        onkyoLock.lock()
        try {
            logDebug("onkyo.rules", onkyoNETList0.state.toString())
            postUpdate(onkyoNETSel0, onkyoNETList0.state.toString.substring(3))             
            postUpdate(onkyoNETSel1, " ")               
            postUpdate(onkyoNETSel2, " ")               
            postUpdate(onkyoNETSel3, " ")               
            postUpdate(onkyoNETSel4, " ")               
            postUpdate(onkyoNETSel5, " ")               
            postUpdate(onkyoNETSel6, " ")               
            postUpdate(onkyoNETSel7, " ")               
            postUpdate(onkyoNETSel8, " ")               
            postUpdate(onkyoNETSel9, " ")               
        } finally {
           onkyoLock.unlock()
        }
end
rule "Update List 1 Item"
    when
        Item onkyoNETList1 received update
    then
    createTimer(now.plusSeconds(1))[|
        onkyoLock.lock()
        try {
            logDebug("onkyo.rules", onkyoNETList1.state.toString())
            postUpdate(onkyoNETSel1, onkyoNETList1.state.toString.substring(3))             
        } finally {
           onkyoLock.unlock()
        }
    ]
end
rule "Update List 2 Item"
    when
       Item onkyoNETList2 received update
    then
    createTimer(now.plusSeconds(1))[|
        onkyoLock.lock()
        try {
            logDebug("onkyo.rules", onkyoNETList2.state.toString())
            postUpdate(onkyoNETSel2, onkyoNETList2.state.toString.substring(3))             
        } finally {
           onkyoLock.unlock()
        }
    ]
end
rule "Update List 3 Item"
    when
        Item onkyoNETList3 received update
    then
    createTimer(now.plusSeconds(1))[|
        onkyoLock.lock()
        try {
            logDebug("onkyo.rules", onkyoNETList3.state.toString())
            postUpdate(onkyoNETSel3, onkyoNETList3.state.toString.substring(3))             
        } finally{
           onkyoLock.unlock()
        }
    ]
end
rule "Update List 4 Item"
    when
        Item onkyoNETList4 received update
    then
    createTimer(now.plusSeconds(1))[|
        onkyoLock.lock()
        try {
            logDebug("onkyo.rules", onkyoNETList4.state.toString())
            postUpdate(onkyoNETSel4, onkyoNETList4.state.toString.substring(3))             
        } finally {
           onkyoLock.unlock()
        }
    ]
end
rule "Update List 5 Item"
    when
        Item onkyoNETList5 received update
    then
    createTimer(now.plusSeconds(1))[|
        onkyoLock.lock()
        try {
            logDebug("onkyo.rules", onkyoNETList5.state.toString())
            postUpdate(onkyoNETSel5, onkyoNETList5.state.toString.substring(3))             
        } finally {
           onkyoLock.unlock()
        }
    ]
end
rule "Update List 6 Item"
    when
        Item onkyoNETList6 received update
    then
    createTimer(now.plusSeconds(1))[|
        onkyoLock.lock()
        try {
            logDebug("onkyo.rules", onkyoNETList6.state.toString())
            postUpdate(onkyoNETSel6, onkyoNETList6.state.toString.substring(3))             
        } finally {
           onkyoLock.unlock()
        }
    ]
end
rule "Update List 7 Item"
    when
        Item onkyoNETList7 received update
    then
    createTimer(now.plusSeconds(1))[|
        onkyoLock.lock()
        try {
            logDebug("onkyo.rules", onkyoNETList7.state.toString())
            postUpdate(onkyoNETSel7, onkyoNETList7.state.toString.substring(3))             
        } finally {
           onkyoLock.unlock()
        }
    ]
end
rule "Update List 8 Item"
    when
        Item onkyoNETList8 received update
    then
    createTimer(now.plusSeconds(1))[|
        onkyoLock.lock()
        try {
            logDebug("onkyo.rules", onkyoNETList8.state.toString())
            postUpdate(onkyoNETSel8, onkyoNETList8.state.toString.substring(3))             
        } finally {
           onkyoLock.unlock()
        }
    ]
end
rule "Update List 9 Item"
    when
        Item onkyoNETList9 received update
    then
    createTimer(now.plusSeconds(1))[|
        onkyoLock.lock()
        try {
            logDebug("onkyo.rules", onkyoNETList9.state.toString())
            postUpdate(onkyoNETSel9, onkyoNETList9.state.toString.substring(3))             
        } finally {
           onkyoLock.unlock()
        }
    ]
end
```