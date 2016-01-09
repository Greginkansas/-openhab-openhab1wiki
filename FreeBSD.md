# Installation on FreeBSD

NB Small warning, this is only for the more technically versed as it requires patching and recompiling openHAB

There are currently two known problems with running openHAB on FreeBSD out of the box. 

1. The serial package that is used by openHAB does not support FreeBSD. Fortunately it is not difficult to compile it, but it does require a number of steps.
2. There is a "bug" (missing feature, the jury is out on that :)) in that the FreeBSD IP stack does not support mapping IPv4 multicast addresses inside IPv6 addresses. This problem occurs when setting up multicast, and impacts quite a bit of functionality. The workaround for this problem is fortunately very simple.

This procedure was tested on FreeBSD 10.2 with openHAB 1.8.0.

1. Install openjdk8 and git on FreeBSD:
     pkg install openjdk8 git
1. Install maven via ports (otherwise it's going to download jdk7 for some reason)
   1. cd /usr/ports/devel/maven33
   1. make install clean
2. Get the openHAB sources for 1.8.0, see also [[IDE Setup|IDE-Setup]]. You can skip the parts about setting up eclipse, just get the sources and use maven to compile. Pay special attention to also cleaning before building the sources, this is required every time! Say the location for the openhab sources is $(OHSRC)
3. Get the nrjavaserial sources from <https://github.com/NeuronRobotics/nrjavaserial>
3. You'll also need gradle, so install that as well, again as port: 
  1. cd /usr/ports/devel/gradle
  1. make install clean
4. Apply the patches from the thread <https://groups.google.com/forum/?pli=1#!topic/openhab/fLAs5NdLwpw>. I haven't installed the patches automatically (as described in the thread), but applied them one by one by hand. It may still work, but YMMV.
5. Build nrjavaserial (basically do 'make freebsd')
6. Get the resulting library (for version 3.11.0) from build/libs/nrjavaserial-3.11.0.jar and copy it to $(OHSRC)/bundles/io/org.openhab.io.transport.serial/lib/
7. Change the version number in $(OHSRC)/bundles/io/org.openhab.io.transport.serial/build.properties and $(OHSRC)/bundles/io/org.openhab.io.transport.serial/META-INF/MANIFEST.MF
8. cd $(OHSRC)
9. mvn clean
9. mvn package
9. wait 10 minutes
9. The new distribution files are in $(OHSRC)/distribution/target
9. Install openHAB as usual, see also [[Manual installation|Linux---OS-X#manual-installation-alternative-approach]]
9. Edit start.sh and add the following define: -Djava.net.preferIPv4Stack=true
9. Edit configurations/openhab.cfg and set the serial port to /dev/ttyU0 (or adapt if your usb device shows up on a different device)

# Regarding the multicast problem

For those interested in more background, it seems the FreeBSD IP stack does not support mapping IPV4 multicast addresses inside IPV6 addresses. Here is a link to the corresponding bug report:

https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=193246

It turns out to be a non-trivial amount of work to fix this, and the use case is apparently not entirely clean, so resolution of the bug fix is still pending. Fortunately there is also a work-around. Adding the following define to start.sh makes it work:

-Djava.net.preferIPv4Stack=true

