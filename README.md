# Mirawatt TED1000
Moving the TED1000 scripts to github, and automating the startup.

on cantor: `/home/daniel/Documents/NetBeansProjects/green/scalr-utils/` points to
svn repo at `https://snookr.googlecode.com/svn/trunk/green/scalr-utils`.
On dirac it is at `/Users/daniel/Downloads/green/scalr-utils`

    
The plan is to move to upstart;

*   detect USB device
*   capture job
*   summarize job
*   push to imetrical
*   monitor (local shell)
*   move the code

## Detect usb device
_Note: sheeva's console usb port is also a FTDI device, watch it!_

I need to write a `udev` rule to assign a known name to the device.

    root@cantor:~# lsusb|grep -i serial
    Bus 003 Device 002: ID 067b:2303 Prolific Technology, Inc. PL2303 Serial Port
    Bus 002 Device 002: ID 0403:6001 Future Technology Devices International, Ltd FT232 USB-Serial (UART) IC
	find /sys -name dev|grep -i ttyusb
	/sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1:1.0/ttyUSB1/tty/ttyUSB1/dev
	/sys/devices/pci0000:00/0000:00:1d.1/usb3/3-2/3-2:1.0/ttyUSB0/tty/ttyUSB0/dev	
    
The FTDI device is TED's. (the other PL2303 is the current cost device).

root@cantor:/etc/udev/rules.d# udevadm info -a -p $(udevadm info -q path -n TED1K) | egrep -i "ATTRS{serial}|ATTRS{idVendor}|ATTRS{idProduct}" -m 3
    ATTRS{idVendor}=="0403"
    ATTRS{idProduct}=="6001"
    ATTRS{serial}=="A7004VwU"

So looks like all I need to make `/dev/ttyTED1k`,`/dev/ttyCRNTCOST` is a rule like


    # cat 10-ftdi-ted1k.rules 
    BUS=="usb", SYSFS{idVendor}=="0403", SYSFS{idProduct}=="6001", NAME="TED1K"

    
    kernel=="ttyUSB*", SYSFS{idVendor}=="0403", SYSFS{idProduct}=="6001", SYMLINK="TED1K"
    
 The one for current cost would be:
    
    kernel=="ttyUSB*", SYSFS{idVendor}=="067b", SYSFS{idProduct}=="2303", SYMLINK="CRNTCOST"

### References

*   [Rule example for FTDI](http://aeturnalus.com/robotics/mapping-ftdi-to-files-with-udev/)
*   [Udev rule writing](http://www.reactivated.net/writing_udev_rules.html#ownership)
*   [Thread that brought me there](http://fixunix.com/ubuntu/494414-serial-ports-ttyusb%7B0-9%7D.html)

 
## Python dependencies

*   ReadTEDNative needs pyserial, and mysqldb both seem to be available as packages; check from sheeva   
*   Summarize need mysqldb

### Installing pyserial (using ubuntu package)
Under lucid (10.04), python-serial a.k.a pyserial is at version 2.3-1

    apt-cache show python-serial
    dpkg -l python-serial

Under Oneiric (11.10) it is 2.5-2.1. Shoud test to see that it works.
We should also see if this works under sheeva


## Create an upstart script
Test on aria,miraplug001 with modified ReadTEDNative not talking to mysql;
