**Use symlinks if you use more than one USB port.**  If you have more than one USB device (e.g. a Zwave dongle and an RFXCOM dongle, the USB name will change every time you reboot.  To prevent this, 
create or add to existing file (`/etc/udev/rules.d/50-usb-serial.rules`) a rule like the following:

    SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{product}=="RFXrec433", SYMLINK+="USBrfxcom", GROUP="dialout", MODE="0666" 
    SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", SYMLINK+="USBzwave", GROUP="dialout", MODE="0666"
    SUBSYSTEM=="tty", ATTRS{idVendor}=="0658", ATTRS{idProduct}=="0200", SYMLINK+="USBzwave", GROUP="dialout", MODE="0666"
    # RFXtrx433
    SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{product}=="RFXtrx433", SYMLINK+="ttyUSBrfxtrx", GROUP="dialout", MODE="0666"
    # Plugwise
    SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", SYMLINK+="ttyUSBplugwise", GROUP="dialout", MODE="0666"


The first entry corresponds to the RFXtrx433, the second one to the Aeotec ZStick Gen2 and the last one to the Aeotec ZStick Gen5. You can copy the entries that suit your case or all of them, you won't find any problem unless you add two ZWave USB plugs to the same computer.

If you are going to use any other USB device, you can get the IdVendor, product, and IdProduct, running (for USB0, USB1, ACM0 etc) `sudo udevadm info --attribute-walk --path=/sys/bus/usb-serial/devices/ttyUSB0`. There you can find IdVendor, product, and IdProduct. Replace these IDs in the rule and save the file. Now your stick can be referenced in openHAB configuration as `/dev/USBzwave`. You will also need to add the property to the Java command line by adding the following (device names delimited by `:` ) to the file `/etc/init.d/openhab` in the 'JAVA_ARGS' section with your device names substituted:

    -Dgnu.io.rxtx.SerialPorts=/dev/USBrfxcom:/dev/USBzwave

Note: If you are using the apt-get installation method, you should add the previous line to the 'JAVA_ARGS' section of the '/etc/default/openhab' file.

Note: On Linux; the extra Java command-line property is _not_ required if you choose a symlink name that matches a standard Linux comm port prefix and ends with a combination of numerics + non-letters. E.g. "ttyUSB-9999".