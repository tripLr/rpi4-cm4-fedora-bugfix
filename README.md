# rpi4-cm4-fedora-bugfix
This is the steps I went thru to fix the boot up bug in fedora on RPI4 and CM4
Along the way I deciced to simplify the entire process and take notes in my debuggng and install processes

### My steps to working thru Bugs in fedora server boot problems on RPI installations

    Installation instructions for installing fedora on RPI4 are different from installing on a CM4
    Mostly this is the difference in the bootloader and how to access the CM4 with the bootloader and mass-storage-tool boot image

  1.   Flashing the RPI4 either the sdcard or usb device such as a thumb drive or usb-sata device are essentially the same, if you have a USB-SDCARD reader
       The RPI Imager will see the USB device in in windows, mac, or other RPI computer Operating system. Flashing this method is easy.
       
  2.  The CM4 without EMMC (400x00 or 410x00 series x=ram size ) may boot with a sd card.
      The CM4 with EMMC I have (410800 and 410832) will boot to emmc,nvme or usb, but not sdcard, due to the emmc is on the same hardware traces path as the sdcard reader )

Hardware Tested
'''
rpi4b 8 gb wifi
cm4 8gb 32 gb emmc wifi (4108032) on cm4 pi-developer board
cm4 8gb 32 gb emmc wifi (4108032) on waveshare router board
cm4 8gb 32 gb emmc wifi (4108032) on waveshare RPI nvme board
cm4 8gb wifi nvme       (4108000) on all above 
'''

# Main bug, no shell on video screen , no video after setup
Fedora Versions tested
'''
* testing fedora 38 server
fedora 38 minimal
fedora 38 core os bare metal
fedora 39 beta server
'''
### !! Note !! : this is not the method recommended by Fedora to install, but to use the arm-image-installer
   see below in the writing and debugging sections
    
   wiki : https://fedoraproject.org/wiki/Architectures/ARM/Installation
   source : https://pagure.io/arm-image-installer/tree/main

   Since the majority of people install rasberian first and use it tools, I went there first.
See next section for arm-image-installer tests

# RPI4 Installation section 
    1.Install Imager from RPI
        Web Download https://www.raspberrypi.com/software/
        Latest Ubuntu  https://downloads.raspberrypi.org/imager/imager_latest_amd64.deb
        Latest Windows https://downloads.raspberrypi.org/imager/imager_latest.exe
        Latest Mac     https://downloads.raspberrypi.org/imager/imager_latest.dmg
   
    From Rasberian OS terminal
```
       sudo apt install rpi-imager
```

###Direct Downloads of Rasbery PI os
    
    https://www.raspberrypi.com/software/operating-systems/
  
    2. Install Imager on your computer
    3. Plug in USB Device
    4. Download your OS
    5. Use imager to Flash your USB device
    6. Put USB device into RPI4, turn on and boot

###These are my working RPI images on all the above hardware.  
'''
Rasberian 64 bit via Imager
Manjaro Arm
'''

# CM4 install sources and instructions
     The best web instructions :
        Sources https://www.jeffgeerling.com/blog/2020/how-flash-raspberry-pi-os-compute-module-4-emmc-usbboot
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

# Install usbboot tool for CM4 < Required >
    This is necessary to access and edit the bootloader 
        and access and write to the mass-storage-device on your CM4
        and allows read-write access to the device to find and fix bugs
  
  1. Download usbboot from github source and make tool
```
        git clone --depth=1 https://github.com/raspberrypi/usbboot
        cd usbboot
        make
```
  2. You will notice the binary named rpiboot and several folders
  3. The main tool to access the CM4 emmc and or nvme is the mass-storage-gadget
        sudo ./rpiboot -d mass-storage-gadget
    The rpiboot binary writes the mass-storage-gaget boot binary temporarily to the CM4 bootloader
        then boots that minimal linux bootloader and connects the USB slave to the EMMC and or NVME

# You now should have access the CM4 mass-storage via the slave connection
>    This method is described in the RPI compute module instructions
>   (https://www.raspberrypi.com/documentation/computers/compute-module.html)

>    After rpiboot completes, you will see a new device appear; 
>    this is commonly /dev/sda on a Raspberry Pi but it could be another location 
>    such as /dev/sdb, so check in /dev/ or run lsblk before running rpiboot so you can see what changes.
  
  1. Then your "master" comuter should see the storage available as a device readable by 
     Imager and as block device in your shell
  2. Verify your CM4 is connected by running lsblk in your shell
     Your destop may automount and open the drives. 
  3. This point in the process is the same for accessing that storage to debug it ! See the debug session instructions below.

# Write image to the CM4 mass-storage via Imager

  1. Open the Imager app and write your image of choice to the emmc or nvme storage on the CM4
  2. Unplug the Jumper or turn off the slave switch and boot your CM4

 
# Writing to the CM4 eMMC or via dd (Linux)

>    You now need to write a raw OS image (such as Raspberry Pi OS) to the device. 
>    Note the following command may take some time to complete, depending on the size of the image: (Change /dev/sdX to the appropriate device.)
>
```
    sudo dd if=raw_os_image_of_your_choice.img of=/dev/sdX bs=4MiB
```
>
>    Once the image has been written, unplug and re-plug the USB; you should see two partitions appear (for Raspberry Pi OS) in /dev. In total, you should see something similar to this:
>
>    /dev/sdX    <- Device
>    /dev/sdX1   <- First partition (FAT)
>    /dev/sdX2   <- Second partition (Linux filesystem)
>
>    The /dev/sdX1 and /dev/sdX2 partitions can now be mounted normally.
>
>    Make sure J4 (USB SLAVE BOOT ENABLE) / J2 (nRPI_BOOT) is set to the disabled position and/or nothing is plugged into the USB slave port. Power cycling the IO board should now result in the Compute Module booting from eMMC.

# Post writing to device cleanup
> Fedora Server and others use LVM for the / (root) directory and the entire drive space is not accessable yet.
> The image is not automaticlly adjusted to the size of the drive beyond the inital 8GB image.
> The instructions to do this are quite long and included in the Fedora-Resize-Image.md file included 
> in this archive


# WIP Todo List
# Write image to the CM4 mass storage with the approved arm-image-installer from Fedora 
# Post Writing Fedora Image 
# First boot and Setup 
# Setup root, user, network
# Debugging session Begin
> At this point in the process, the Fedora Blank Screen Bug rears its ugly head. I will describe the other two methods
# Remote access
> Check if IP address exists, ssh access, Cockpit access
# Manually access the storage for remote access
> Set IP
> set ssh
# Manually access
> logs , check
> linux cmdline boot process


# Edit the boot sequence with the boot.conf booloader configurations

    [all]
    BOOT_UART=0
    WAKE_ON_GPIO=1
    POWER_OFF_ON_HALT=0

    # Boot Order Codes, from https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#BOOT_ORDER
    # Try SD first (1), followed by, USB PCIe, NVMe PCIe, USB SoC XHCI then network
    BOOT_ORDER=0xf25641

    # Set to 0 to prevent bootloader updates from USB/Network boot
    # For remote units EEPROM hardware write protection should be used.
    ENABLE_SELF_UPDATE=1

     
   
