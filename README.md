# opwenwrt-zyxel-gs1900-24e

source link: https://scm.linefinity.com/common/openwrt/commit/b515ad10a6e1bd5c5da0ea95366fb19c92a75dea


realtek: add support for ZyXEL GS1900-24E
author	Raylynn Knight <rayknight@me.com>	
	Tue, 17 May 2022 05:15:54 +0200 (23:15 -0400)
committer	Sander Vanheule <sander@svanheule.net>	
	Mon, 6 Jun 2022 10:30:50 +0200 (10:30 +0200)
The ZyXEL GS1900-24E is a 24 port gigabit switch similar to other GS1900
switches.

Specifications
--------------
* Device:    ZyXEL GS1900-24E
* SoC:       Realtek RTL8382M 500 MHz MIPS 4KEc
* Flash:     16 MiB Macronix MX25L12835F
* RAM:       128 MiB DDR2 SDRAM Nanya NT5TU128M8GE
* Ethernet:  24x 10/100/1000 Mbps
* LEDs:      1 PWR LED (green, not configurable)
             1 SYS LED (green, configurable)
             24 ethernet port link/activity LEDs (green, SoC controlled)
* Buttons:   1 "RESET" button on front panel
* Switch:    1 Power switch on rear of device
* Power      120-240V AC C13
* UART:      1 serial header (JP2) with populated standard pin connector on
             the left side of the PCB.
             Pinout (front to back):
             + Pin 1 - VCC marked with white dot
             + Pin 2 - RX
             + Pin 3 - TX
             + PIn 4 - GND

Serial connection parameters:  115200 8N1.

## Installation
------------

### OEM upgrade method:

* Log in to OEM management web interface
* Navigate to Maintenance > Firmware
* Select the HTTP radio button
* Select the Active radio button
* Use the browse button to locate the realtek-rtl838x-zyxel_gs1900-24e-initramfs-kernel.bin file and select open so File Path is updated with filename.
* Select the Apply button. Screen will display "Prepare for firmware upgrade ..." Wait until screen shows "Do you really want to reboot?"then select the OK button
* Once OpenWrt has booted, scp the sysupgrade image to /tmp and flash it:
```
   > sysupgrade -n /tmp/realtek-rtl838x-zyxel_gs1900-24e-squashfs-sysupgrade.bin
```
   it may be necessary to restart the network (/etc/init.d/network restart) on
   the running initramfs image.

### U-Boot TFTP method:

* Configure your client with a static 192.168.1.x IP (e.g. 192.168.1.10).
* Set up a TFTP server on your client and make it serve the initramfs image.
* Connect serial, power up the switch, interrupt U-boot by hitting the
  space bar, and enable the network:
```
   rtk network on
```
* Since the GS1900-24E is a dual-partition device, you want to keep the OEM firmware on the backup partition for the time being. OpenWrt can only boot
  from the first partition anyway (hardcoded in the DTS). To make sure we are manipulating the first partition, issue the following commands:
  ```
  setsys bootpartition 0
  savesys
  ```
* Download the image onto the device and boot from it:
```
    tftpboot 0x84f00000 192.168.1.10:openwrt-realtek-rtl838x-zyxel_gs1900-24e-initramfs-kernel.bin
   bootm
  ```
* Once OpenWrt has booted, scp the sysupgrade image to /tmp and flash it:
```
   sysupgrade -n /tmp/openwrt-realtek-rtl838x-zyxel_gs1900-24e-squashfs-sysupgrade.bin
```
   it may be necessary to restart the network (/etc/init.d/network restart) on the running initramfs image.
   
   ## Un-Install

  In the original ZyXEL firmware, you can select which firmware to boot ("Active" or "Backup").

While OpenWrt can only be installed in the first partition, as already mentioned, you can trigger loading the OEM firmware from the second partition. It's as easy as running:
```
fw_setsys bootpartition 1
```
That's the easiest method to return to stock. You can then overwrite OpenWrt with a stock firmware.
You might need kmod-mtd-rw in order to be able to mount the config partition read/write.

The first-time OpenWrt installation can also be performed from the OEM webpage, no need to attach serial.
