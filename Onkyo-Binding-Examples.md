Here are some examples of configurations for various aspects of Onkyo receivers:

* [NET/USB](Onkyo-Binding-Examples#NetUsb)

#NetUsb

###Menu List Display

When using openHAB, it may be desirable to be able to control your A/V receiver without being able to see the video output of the receiver. This may be the case when you are using extra zone outputs for whole house audio, etc.. Onkyo receivers implement a hierarchical menu structure where each page of menu can contain up to ten items. Older generation (circa 2010) and newer generation (circa 2014) were more Model View Control (MVC) friendly. Older units send menu pages one item at a time, but they send all ten items regardless of whether or not they are all used. Newer units will send the complete menu page in one status update. Units made in between are some what problematic as they send menu pages one item at a time, but they only send as many items as are on the page. This causes problems when one page has say ten items, and the next only has eight. There is no easy way to know that the last two menu items should be cleared as no end of menu indication is given.

My first attempt at solving this problems was to clear all menu items when the first item was received. This didn't work as expected as openHAB executes rules in different threads and there is no guarantee that they will execute in the order they were invoked. This would result in random menu items missing as the rule for the first item executed after they had been updated. I wasn't able to find a good solution for this, but I was able to get things functional by delaying the execution of all rules except the one for the first item.

Onkyo receivers transmit menu items with a string matching the following REGEX:

NLSU[0-9]-[0-9a-zA-Z ]*

There may be other valid characters after the '-'. The 'U' in "NLSU" is for Unicode. Old receivers running old firmware images may have an 'A' here for ASCII.

Onkyo receivers transmit the current cursor position (within the menu page) with a string matching the following REGEX:

C[0-9][CP]

The last character is a 'C' when the cursor moves within the page. The last character is a 'P' when a page change occurs. The NETUSB_OP_UP and NETUSB_OP_DOWN commands can be used to move the cursor up/down within the page. The NETUSB_OP_LEFT and NETUSB_OP_RIGHT commands can be used to scroll the current menu page up/down. The NETUSB_OP_RETURN command can be used to move up in the menu hierarchy. The NETUSB_OP_SELECT command can be used to select the current menu item. The "NLSL[0-9]" command can be used to randomly select a menu item (this is not supported on older models). These elements can be combined with rules and dynamic color to implement a reasonable user interface.

For menu list display a string item is used for each item in the menu (ten strings). These strings capture the raw status updates from the receiver, but are not displayed. There is a rule for each menu item that removes the overhead characters and posts the value to another string item for display. 

Dynamic color is used to indicate the current cursor position. In order to do this an item is used to capture the raw cursor position from the receiver. A rule is run when it changes to strip out the cursor line and post it to another item that is used to control the dynamic color (this simplifies the color rules and is useful later).
  
###Menu List Selection

I have included "NLSL[0-9]" commands on the displayed menu item strings in the hopes that openHAB will eventually support some type of selectable text that sends a command instead of going to a URL. Then the user can just select the menu item on receivers that support "NLSL[0-9]" commands. For older receivers, the NETUSB_OP_SELECT command is used. The user needs to navigate to the desired menu item before this command is sent. Setpoint elements are used for navigation as they provide a more compact arrangement.

###Items


###Sitemap


###Rules


More to come...