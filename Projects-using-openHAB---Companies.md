Collects projects being realised by companies. We'd love to see little videos of your achievements with openHAB.

### New HVAC devices (@Westaflex)

#### Abstract / Description / Dates
We have spent over a year on the project and have accomplished a lot. We've already captured many openHAB features and concepts like the configuration of dynamics of the process (lag time, dead time, etc), as well as trend, monitor and update register values in a Modbus PLC or other device supporting the Modbus/TCP protocol. It currently supports profiles for different devices so you can configure multiple devices or multiple views of the same device. These ranges from core application elements like the tuning contants which can be automatically derived for self-regulating 1st order processes to added noise (uniformly distributed or Gaussian) to the process. At first our Binding only worked with the words from %MW0 to %MW65535 !!! and Dword %DW0 to %DW32767 data type signed and unsigned 16/32 bit integer and 32 bit float. 

#### Screenshots
![](http://media-cache-ec0.pinimg.com/originals/d8/ad/e5/d8ade547f4349a6d1f13e65503ed368c.jpg)
![](http://media-cache-ec0.pinimg.com/originals/5b/aa/2b/5baa2b67f1882964562b6024ef133846.jpg)


#### How did openHAB help
We could soon establish a Demo of intelligent house functionality at first able to work with any device supporting Modbus IP/RTU, Modbus RTU with bluetooth. This HVAC environment takes measurements of a number of temperature values within the house, controls lighting, door mechanisms and received a live video feed from an in-house video camera. The system was created with the purpose of demonstrating the experience of our development team to convert low-level operating and control protocols (CAN bus, ModBus) into the user-friendly higher-level openHAB Binding. Westaflex offers the full range of system, Linux and embedded software development services. Our WAC systems use default modbus data provided by Arduino barebones e.g. a Debian Rasberry server kit all year long.

Thanks to openHAB our WAC Binding already had a solid backend for communication b/n devices that simulates a MODBUS/TCP client, saves and retrieves connection settings (profiles), as well as the creation of custom data lists (hmi-like) and how they interact. This let us READ and WRITE data from field instrument thro' MODBUS interface. Because of its open transparent RESTful web service Interface the Binding was easily implemented into our existing automation solutions.
#### Credits
We believe we have successfully delivered this openHAB project with our expertise in software engineering, agile methods, quality assurance and most importantly, our passion of making the world a better place. 

#### External, Press, Video links
http://data.fir.de/projektseiten/win-d/

#### Contact

<tbd>

***

### <project name>
#### Abstract / Description / Dates
#### Screenshots
#### How did openHAB help
#### External, Press, Video links
#### Contact