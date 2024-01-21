  _____  _____ _____        _____ _____ __  __ __  __ 
 |_   _|/ ____|  __ \      / ____|_   _|  \/  |  \/  |
   | | | (___ | |__) |____| (___   | | | \  / | \  / |
   | |  \___ \|  ___/______\___ \  | | | |\/| | |\/| |
  _| |_ ____) | |          ____) |_| |_| |  | | |  | |
 |_____|_____/|_|         |_____/|_____|_|  |_|_|  |_|
                                                                                                                                                      
     ZZJ ISP-SIMM v1.0
  by zigzagjoe (https://github.com/ZigZagJoe/ISP-SIMM_public)

-----------------------------------------------------
| Description                                       |
-----------------------------------------------------

This is a universal ROM SIMM for Apple Macintosh II and Quadra series computers, 
with a built-in USB programmer. It is intended to support rapid development 
of bare-metal code on Macintosh platforms.

This module has 16MB of total storage, exposed to the host 8MB at a time. 
Which half ("bank") the host 'sees' from is controlled by a switch on the module.

-----------------------------------------------------
| Basic operation                                   |
-----------------------------------------------------

1. Using appropriate ESD precautions, install the ROM SIMM in your Macintosh
2. Connect Micro USB cable to the module and your computer.
3. Copy a UF2 file to the ISP-SIMM drive to immediately write it to the active bank
4. Power on your Macintosh once the LED stops blinking and turns (dim) green

See following sections for how to convert .ROM to .UF2 format.

-----------------------------------------------------
| Notes/FAQ                                         |
-----------------------------------------------------

* This module is suitable for:
    Macintosh II series systems with a ROM slot
    Macintosh SE/30
    Quadra systems with a ROM slot (You may need to install a slot)
 
* Quadra's onboard ROMs are disabled if module is installed
    Remove the module to boot using onboard ROMs.
 
* USB does not need to be connected for the ROM module to function.
* USB may be plugged in or unplugged at any time regardless of Macintosh power state
 
* It is normal for the ISP-SIMM drive to briefly disappear and reappear after writing an image. 
    On some OS, this may complain that the drive was improperly ejected. This can be ignored.

* Select which bank the Macintosh will boot from using the switch on the top of the module
    You have two fully independent images that can accept 8MB images. 
    Do not change this switch while the mac is running or it will (almost certainly) crash.

* Do not use this module with any 3rd party ROM programmer devices
    It may cause damage to the module or the programmer.
	
* Apple ROM Flash utility and any other software flash utilities are not supported

*****************************************************
******************** IMPORTANT **********************
*****************************************************
* Any read of a UF2 file, or copy of UF2 file to the drive will disable the Macintosh's access to ROM. 
* This WILL crash the mac OR hold it in reset if the RESET_OUT connection is connected. 
*
* Once the operation finishes, the macintosh will be allowed to read again.
* The mac will automatically restart if the RESET_OUT has been connected.
*
* If you find the macintosh immediately crashes / resets when connecting USB, 
* check your virus scanner is not trying to scan the ISP-SIMM drive.
*****************************************************

-----------------------------------------------------
| UF2 File format                                   |
-----------------------------------------------------

This is the format used for reading and writing ROM images from the ISP-SIMM.
It is particularly suitable for use with microcontrollers due to self-contained data blocks.
It is an open format developed by Microsoft: https://github.com/microsoft/uf2/tree/master

--- Converting files ---

Two scripts have been provided with appropriate predefined settings. 

# Bin2UF2.PS1 (For Windows) 

Right-click, run with powershell, pick your input ROM file(s), and select your output directory.
It also supports command line arguments for batch processing / scripting: -Files and -Outdir
Report bugs to me.

# uf2conv.py (For Macintosh, Linux, or windows with Python3 installed) 

This will prompt interatively for a file (windows only), or accept a file on the command line.
Note: by default it will look for the ISP-SIMM drive and immediately begin flashing it if found. 
Please see here for more details: https://github.com/microsoft/uf2/blob/master/utils/uf2conv.md

# UF2 parameters for other tools

Model: ZZJ ISP-SIMM
Board-ID: ISP-SIMM-V1.0
UF2 Start Address: 0x0000 
UF2 Family ID: 0x10C68030

-----------------------------------------------------
| LED modes                                         |
-----------------------------------------------------

Dim Green               Power on, idle
Dim Red                 in Pico Bootloader mode (ROM is disabled)
Bright Red              Write failure occurred (invalid file or write was aborted)
Bright Green blinking   Read in progress
Bright Red blinking		Write in progress, errors have occurred
Green + Red blinking    Write in progress

Bright green solid / Green and Red solid / LEDs off: 
  host has stopped writing/reading, a time out will occur shortly.

-----------------------------------------------------
| USB terminal                                      |
-----------------------------------------------------

Open the USB serial port using your terminal software of choice, baud rate does not matter.
By default, log entries will be shown here as they occur.

Press ~~~ (3 tildes) and WAIT until "COMMAND?" is displayed before entering a command.
All other input will be forwarded to the UART header onboard.

--- Commands ---

echo 0|1     disable/enable debug logging to USB. 
  Required for running other protocols over USB-Serial
  This setting will be saved and used at next power-on of SIMM.

baud n       set serial uart baud rate to n
  Immediately change the uart's baud rate (this does no change anthing on the USB serial port)
  This setting will be saved and used at next power-on of SIMM.

bank a|b|c   set bank a|b or (c)lear to use switch's setting 
  Use to specify which bank is written when dragging a UF2 file to the drive, or what BANK_CUR.UF2 reads back.
  This will override the physical switch until the module is reset or "bank c" is issued

refresh      refresh drive contents to OS
  Causes the USB device to briefly eject and re-insert its media. 
  This forces most OS to update the contents of files and is performed automatically on writes.

bootloader   reboot to pico uf2 bootloader
  Immediately reboots to the pico's UF2 bootloader mode
  
resetmac     resets macintosh (requires RESET_OUT connection)
  Immediately resets the macintosh using the RESET_OUT connection.
  Ignored while ROM flashing or read operations are in progress.
   
showlog      prints recent log entries
  requires debug logging to usb to be enabled!!
  
-----------------------------------------------------
| Firmware updates                                  |
-----------------------------------------------------

Firmware is posted on github, at the link at the top of this document.
Please report issues there.

--- How to update ---

1. Shut down the mac and disconnect module from computer. 
2. Hold the "BOOTSEL" switch on the back of the module
3. Connect the module to the computer.
4. You should see a RPI-RP2 drive appear on the computer. You may release the BOOTSEL switch.
5. Drag the .uf2 file obtained from github to this drive.
6. The RPI-RP2 drive will disappear, and the ISP-SIMM drive should reappear.
     Your computer may complain about unclean ejection of drive. This can be ignored.
7. Verify new firmware version by checking INFO_UF2.TXT

--- Alternate method to enter bootloader ---

Issue the bootloader command from the serial terminal, then continue from step 4.

-----------------------------------------------------
| Troubleshooting                                   |
-----------------------------------------------------

--- Checking the log ---

Open ISP-SIMM.log in text editor of choice. This contains essential troubleshooting information.

NOTE: This log file DOES NOT UPDATE. This is an unavoidable limitation of how I present storage to the host.
To update this log: issue a "refresh" command via serial, write an image to the drive, or unplug/replug USB.

I recommend that you use serial log for realtime troubleshooting.

Note: More recent entries will begin overwriting old ones at the top of the log file.

---------

No boot
    Verify the ROM is correctly seated. 
    If your ROM socket is old or dirty, the module may need some pressure to correctly make contact. 
    Make sure you're writing a ROM file that is compatible with your Macintosh
    Verify the LED is DIM green after write completes (success)
    Check the log for more information.
	
USB does not connect
	Try a different cable or use the cable supplied.
	Reflash firmware to SIMM

Write errors 
    Try again.
    Try connecting USB directly to computer
    Try a different computer
    As always, check the log.
 
Mac crashes / resets unexpectedly or when USB is plugged in
    Verify antivirus software is not scanning the ISP-SIMM drive
  
BANK_A.UF2 OR BANK_B.UF2 are 0 bytes in size
    A write error occurred the last time you were flashing this bank. 
    Please try again, and consult the log if issues continue.
 
BANK_CUR.UF2 is always 16MB in size
	Due to limitations in how the USB mass storage works, it is not possible to read out anything but the full bank.
	
Feel free to reach out to me on github or 68kmla with log files if you continue to have issues.

 ###################################################################################
#####################################################################################
###    / \     Scary message:                                                     ###
###   /   \                                                                       ###
###  /  !  \   Advanced functionality past this point. Proceed at your own risk!  ###
### /_______\  I take no responsibility if you damage your SIMM or Macintosh.     ###
#####################################################################################
 ###################################################################################
 
-----------------------------------------------------
| RESET_OUT header                                  |
-----------------------------------------------------

OPTIONAL. This allows the SIMM to automatically reset the Mac and hold it in reset 
while SIMM read/write operations are in progress. Open collector output, 5V logic safe. 

Connect to the a RESET pin (such as found on the reset button) or on a header (ie. Pin A12 of SE/30 PDS).

Make very certain you are connecting this correctly or you WILL cause damage 
to either the SIMM or your macintosh.

You can test by plugging the wire into the appropriate location on your logic board, 
and then connecting it to ground using a 100 - 1000 ohm resistor. 

Your system should reset and not boot until disconnected.

-----------------------------------------------------
| Serial (UART) header                              |
-----------------------------------------------------

The ISP-SIMM has a hardware UART exposed on the serial header. 

All UART data received is forwarded to USB serial, and all input to the USB serial 
is sent to the UART connected device. See above for configuration notes.

This is a SM04B-SURS type connector. Pin 1 is designated by a small triangle.

Pin 1: GND
Pin 2: Pico RX (Device TX)
Pin 3: Pico TX (Device RX)
Pin 4: +5V Power

*****************************************************
******************** IMPORTANT **********************
*****************************************************
* UART TX/RX pins are 3V logic. They are NOT 5V tolerant!!! 
* DO NOT connect +5V or 5V logic to TX or RX! You WILL damage your SIMM.
* A level converter (MAX3232) is required to connect to Macintosh.
*
* I am sorry for providing +5V power and not +3.3V :(
*****************************************************

-----------------------------------------------------
| External USB                                      |
-----------------------------------------------------

USB pads are exposed on the bottom of the module so you may solder a USB cable (at your own risk!)

On the bottom of the Pico module: D+ is nearest to the +sign
On the bottom of the Pico module: D- is nearest to the -sign
+5V is immediately next to the +, over the PWRIN label
GND is immediately next to the -, over the PWRIN label

_____________________________________________________________
          \        _______    _______        /       
           \      |       |  |       |      /            ____  
            \     |   D+  |  |   D-  |     /            /
             \    |_______|  |_______|    /            /
              \__________________________/            /
                    ______    ______                 (    
             |     /      \  /      \                (    
           -----  |  +5V   ||  GND   |  ------        \ 
             |     \______/  \______/                  \
                                                        \____
                  P    W    R   I   N                    
				  
				  
-----------------------------------------------------
| External Bank Select Switch                       |
-----------------------------------------------------

You can use an external bank select switch so long as you leave the onboard switch in the B position.
Solder leads to the two pins that are shorted when the switch is in the A position.
