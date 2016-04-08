Here are some examples of configurations for various aspects of Onkyo receivers:

* [NET/USB](Onkyo-Binding-Examples#NetUsb)

#NetUsb

###Menu List Display

When using openHAB, it may be desirable to be able to control your A/V receiver without being able to see the video output of the receiver. This may be the case when you are using extra zone outputs for whole house audio, etc.. Onkyo receivers implement a hierarchical menu structure where each page of menu can contain up to ten items. Older generation (circa 2010) and newer generation (circa 2014) were more Model View Control (MVC) friendly. Older units send menu pages one item at a time, but they send all ten items regardless of whether or not they are all used. Newer units will send the complete menu page in one status update. Units made in between are some what problematic as they send menu pages one item at a time, but they only send as many items as are on the page. This causes problems when one page has say ten items, and the next only has eight. There is no easy way to know that the last two menu items should be cleared as no end of menu indication is given.

My first attempt at solving this problems was to clear all menu items when the first item was received. This didn't work as expected as openHAB executes rules in different threads and there is no guarantee that they will execute in the order they were invoked. This would result in random menu items missing as the rule for the first item executed after they had been updated. I wasn't able to find a good solution for this, but I was able to get things functional by delaying the execution of all rules except the one for the first item.

Onkyo receivers transmit menu items with a string matching the following REGEX:

NLSU[0-9]-[0-9a-zA-Z ]*

There may be other valid characters after the '-'. The 'U' in "NLSU" is for Unicode. Old receivers running old firmware images may have an 'A' here for ASCII.
  
###Menu List Selection
    