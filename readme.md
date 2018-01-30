`Systemd` Files for Nut UPS Server/Monitor
==========================================

This repository contains the files I created to enable my server to monitor its own UPS, attached
via USB, using scripts created to enable it to work smoothly with `systemd`. These scripts are a
variant of those available from the instructions on
<https://www.kepstin.ca/blog/networkupstoolsnutandsystemd/>.

The string `3B1234X98765` appears in several places. It is a placeholder for the serial number of
your UPS.

## How It Works

The `udev` rules script creates a device symlink at `/dev/ups/3B1234X98765` pointing to the actual
USB device at its appropriate position in the bus heirarchy. This gives us a permanent reference
that won't change based on which USB port the UPS is attached to. On my system this appears as:

 - `/dev/ups/3B1234X98765 -> /dev/bus/usb/003/004`

The `systemd` components are:

 - `dev-ups-3B1234X98765.device`: device identification file
 - `nut-driver@.service`: tells the system how to start `upsdrvctl` for each device
 - `nut-monitor.service`: tells the system how to start `upsmon` for all devices
 - `nut-server.service`: tells the system how to start `upsd` for all devices
 
The Nut server connects to one or more Nut drivers to distribute access to the state of the UPSs.
The Nut monitor connects to the server and monitors the state of the UPS, ready to initiate a
shutdown if the power levels get too low, for example.

## Customizing for Your System

To find your device's serial number, run `sudo lsusb -v` and search for your UPS device in the
list. On my machine, this looks like so:

	Bus 003 Device 004: ID 051d:0002 American Power Conversion Uninterruptible Power Supply
	Device Descriptor:
	  bLength                18
	  bDescriptorType         1
	  bcdUSB               2.00
	  bDeviceClass            0 (Defined at Interface level)
	  bDeviceSubClass         0 
	  bDeviceProtocol         0 
	  bMaxPacketSize0        64
	  idVendor           0x051d American Power Conversion
	  idProduct          0x0002 Uninterruptible Power Supply
	  bcdDevice            0.90
	  iManufacturer           1 American Power Conversion
	  iProduct                2 Back-UPS NS1080G FW:914.L2   .D USB FW:L2    
	  iSerial                 3 3B1234X98765  
	  bNumConfigurations      1
	  Configuration Descriptor:
		bLength                 9
		bDescriptorType         2

Note the item for `iSerial`, which shows the serial number. The name of the file
`/etc/systemd/system/dev-ups-3B1234X98765.device` should be renamed based on your serial number.
Likewise, there's a reference to the serial number inside the following files:

 - `/etc/nut/ups.conf` (twice)
 - `/etc/nut/upsmon.conf`
 - `/etc/systemd/system/nut-server.service`

I've included my `/etc/nut` files for convenience. Note that it contains a password in two places:
`upsd.users` and `upsmon.conf`. You'll want to change that password. The files should be marked
readable **only by root** so that this password is kept secure, i.e. `chmod 600`.
