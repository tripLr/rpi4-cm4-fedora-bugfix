# rpi4-cm4-fedora-bugfix
This is the steps I went thru to fix the boot up bug in fedora on RPI4 and CM4
Along the way i deciced to simplify and take notes in my debuggng and install processes

My steps to working thru Bugs in fedora server boot problems on RPI installations

Installation instructions for installing fedora on RPI4 are different from installing on a CM4
     Mostly this is the difference in the bootloader and how to access the CM4 with the bootloader and mass-storage-tool boot image

  A:   Flashing the RPI4 either the sdcard or usb device such as a thumb drive or usb-sata device are essentially the same, if you have a USB-SDCARD reader
       The RPI Imager will see the USB device in in windows, mac, or other RPI computer Operating system. Flashing this method is easy.
       
  B:  The CM4 without EMMC (400x00 or 410x00 series x=ram size ) may boot with a sd card.
      The CM4 with EMMC I have (410800 and 410832) will boot to emmc,nvme or usb, but not sdcard, due to the emmc is on the same hardware traces path as the sdcard reader )

Hardware Tested
rpi4b 8 gb wifi
cm4 8gb 32 gb emmc wifi (4108032) on cm4 pi-developer board
cm4 8gb 32 gb emmc wifi (4108032) on waveshare router board
cm4 8gb 32 gb emmc wifi (4108032) on waveshare RPI nvme board
cm4 8gb wifi nvme       (4108000) on all above 

# Main bug, no shell on video screen , no video after setup
Fedora Versions tested
fedora 38 server
fedora 38 minimal
fedora 38 core os bare metal
fedora 39 beta server

# *** note : this is not the method recommended by Fedora to install, but to use the arm-image-installer
       
   wiki : https://fedoraproject.org/wiki/Architectures/ARM/Installation
   source : https://pagure.io/arm-image-installer/tree/main

   Since the majority of people install rasberian first and use it tools, I went there first.
See next section for arm-image-installer tests

# RPI4 Installation section 
1. Install Imager from RPI
     Web Download https://www.raspberrypi.com/software/
   Latest Ubuntu  https://downloads.raspberrypi.org/imager/imager_latest_amd64.deb
   Latest Windows https://downloads.raspberrypi.org/imager/imager_latest.exe
   Latest Mac     https://downloads.raspberrypi.org/imager/imager_latest.dmg
   From Rasberian OS terminal
   </code>
      sudo apt install rpi-imager
   <code/>

   Direct Downloads of Rasbery PI os
   https://www.raspberrypi.com/software/operating-systems/
  
2. Install Imager on your computer
3. Plug in USB Device
4. Download your OS
5. Use imager to Flash your USB device
6. Put USB device into RPI4, turn on and boot

These are my working RPI images on all the above hardware.  Rasberian 64 bit via Imager menu
  Manjaro Arm

# CM4 install sources and instructions
     The best web instructions :
    Sources :   https://www.jeffgeerling.com/blog/2020/how-flash-raspberry-pi-os-compute-module-4-emmc-usbboot
               https://www.raspberrypi.com/documentation/computers/compute-module.html

NOTE the instructions dont explicitly say whats needed in this repo to get access to the CM4
the tool you download from github does 3 things.

  a. Update the CM4 Bootloader
  b. Provides a way to edit the boot sequence in the bootloader , which is NECESSARY to setting your CM4 for the job you want to do.
  c. sends a custom boot image to the CM4 and boots it to turn the USB C slave / power connector into a USB drive with access to the EMMC and or the 
      NVME if you have an NVME attached to a board that has a PCIE-EMMC controller

# CM4 carrier board notes
   
   The Stock CM4 carrier board from RPI has a jumper to program the CM4 bootloader and access the usb and emmc mass storage devices
    This board has a non-powered micro-usb connector for flashing and accessing storage only.
  Some boards have a switch that goes between a slave/run position for the usb and may accept usb power in the run position to power the board
      My waveshare nvme boards have this option
   Some Carrier boards automaticly turn on slave mode with a connection to the usb-c from another computer,
      and have a seperate 12v or other power connection, ie waveshare router board

# Install and access your CM4 on your carrier board

  1. Install your CM4 on the CM4 carrier board, set the jumper / switch to slave mode
  2. Connect a usb cable to the CM4 usb-c or micro-usb slave port from your master ( running ) computer
  3. Power on the CM4 board if external power is needed,
  4. Status light should show on, or bootloader ready for flashing emmc or nvme or both 

# Install usbboot tool for CM4
    This is necessary to access the bootloader and mass-storage-device on your CM4
  
  1. Download usbboot from github source and make tool
     instructions : 
    
     
   
