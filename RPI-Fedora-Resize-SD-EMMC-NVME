# Rpi - CM4 Fedora FileSystem Resize
### Source https://raspberrypi.stackexchange.com/questions/133150/how-to-expand-the-volume-of-fedora-35-server

##  After writing the Fedora Image to the memory system with the RPi Imager or Linux dd tool,
##  The file system doesnt automaticlly resize the root partition to the entire availbale memory
##   on the Flash memory device, such as a RPi4 SD card, a CM4 EMMC memory or a CM4 NVME drive, 
##      or USB memory or USB drive adapter

Issue: Volume Resize

###Example 
CM4 32GB EMMC 

1. Mount the EMMC using a Boot USB RPI image
   or the rpiboot -d mass-storage-gadget
'''
cd usboot
./rpiboot -d mass-storage-gadget
'''

# Image Eample using lsblk after your device is booted and you see you
   only have a 5.4 GB partiton on your 32 gb card 
```
lsblk
NAME                   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
mmcblk0                179:0    0 29.8G  0 disk 
├─mmcblk0p1            179:1    0  600M  0 part /boot/efi
├─mmcblk0p2            179:2    0    1G  0 part /boot
└─mmcblk0p3            179:3    0  5.4G  0 part 
  └─fedora_fedora-root 253:0    0  5.4G  0 lvm  /
zram0                  252:0    0  7.6G  0 disk [SWAP]
'''

# Access storage with another computer

Using an SD card adapter on another computer
or booting to a usb drive on your RPI
Or booting up another RPi and isung the mass-storage-gadget

Create an alias called LSBLK to expose all your disk information
'''
alias LSBLK='lsblk -o NAME,SIZE,VENDOR,MODEL,TYPE,FSTYPE,UUID,MOUNTPOINT'
LSBLK
[22:29:49:Sat Nov 04:~/usbboot](master)
[root@triplr-cm4-router:]$LSBLK
NAME         SIZE VENDOR   MODEL                        TYPE FSTYPE      UUID                                   MOUNTPOINT
sdb         29.1G mmcblk0  Raspberry Pi multi-function  disk                                                    
├─sdb1       600M                                       part vfat        6E0C-FE6D                              
├─sdb2         1G                                       part xfs         f9ef8131-4212-4194-bbcb-3f04626f34a4   
└─sdb3       5.4G                                       part LVM2_member 7IeeNH-xPfj-gtdm-q63k-MkND-Mqg0-WRg5Ni 
mmcblk0     29.1G                                       disk                                                    
├─mmcblk0p1
│          457.8M                                       part vfat        14E9-C189                              /boot
└─mmcblk0p2
            28.6G                                       part ext4        d0365520-aedf-4d98-a77b-350122736c3f   /
mmcblk0boot0
               4M                                       disk                                                    
mmcblk0boot1
               4M                                       disk                                                    
zram0       11.4G                                       disk                                                    [SWAP]
'''

Note, I am accessing my CM4 with the mass-storage-gadget 
on my CM4 Router and the device /dev/sdb is the EMMC on the other CM4

### First we need to resize the partition for Physical Volume. 
It's can be done by so many ways but those are newbie like me can follow those steps as following:

# Resize the Partition
## Use fdisk ( risk facor high unless you know what is happening )

fdisk reads write and deletes partitions and partition tables.

Technicly by expading the last partition into the empty space by 
   deleting the partition
     and creating a new partition with the same type and beginning sector
        and using the last sector available

These are the steps
  1. use fdisk to access the partition 
  2. delete the partiton ( this DOES NOT DELETE THE DATA )
  3. make a new partiton in the same beginning but new end 
  4. change the partition type to the SAME LVM code
  5. write to the device

```
sudo fdisk /dev/sdb
:'
Command (m for help): p
Command (m for help): d      <---this deletes the partition select the partition number next
Partition number (1-3, default 3): 3   

Command (m for help): n
Select (default p): p
Partition number (3,4, default 3): 3
First sector (3328000-62521343, default 3328000): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (3328000-62521343, default 62521343): 
Do you want to remove the signature? [Y]es/[N]o: N

Command (m for help): t
Partition number (1-3, default 3): 3
Hex code or alias (type L to list all): 8e

Command (m for help): p
Command (m for help): w
```

# Then we need to expand Physical & Logical Volume as following:
Status: Before Expand

[22:41:57:Sat Nov 04:~/usbboot](master)
[root@triplr-cm4-router:]$LSBLK
NAME         SIZE VENDOR   MODEL                            TYPE FSTYPE      UUID                                   MOUNTPOINT
sdb         29.1G mmcblk0  Raspberry Pi multi-function USB  disk                                                    
├─sdb1       600M                                           part vfat        6E0C-FE6D                              
├─sdb2         1G                                           part xfs         f9ef8131-4212-4194-bbcb-3f04626f34a4   
└─sdb3      27.5G                                           part LVM2_member 7IeeNH-xPfj-gtdm-q63k-MkND-Mqg0-WRg5Ni 
mmcblk0     29.1G                                           disk                                                    
├─mmcblk0p1
│          457.8M                                           part vfat        14E9-C189                              /boot
└─mmcblk0p2
            28.6G                                           part ext4        d0365520-aedf-4d98-a77b-350122736c3f   /
mmcblk0boot0
               4M                                           disk                                                    
mmcblk0boot1
               4M                                           disk                                                    
zram0       11.4G                                           disk                                                    [SWAP]


lsblk
:'
NAME                   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
mmcblk0                179:0    0 29.8G  0 disk 
├─mmcblk0p1            179:1    0  600M  0 part /boot/efi
├─mmcblk0p2            179:2    0    1G  0 part /boot
└─mmcblk0p3            179:3    0 28.2G  0 part 
  └─fedora_fedora-root 253:0    0  5.4G  0 lvm  /
zram0                  252:0    0  7.6G  0 disk [SWAP]
'

Expand: Physical Volume

sudo umount /dev/mmcblk0p3
sudo pvresize /dev/mmcblk0p3 
sudo pvs

Expand: Logical Volume

sudo lvresize -l +100%FREE /dev/fedora_fedora/root
sudo xfs_growfs /dev/fedora_fedora/root
sudo lvs

Status: After Expand

lsblk
:'
NAME                   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
mmcblk0                179:0    0 29.8G  0 disk 
├─mmcblk0p1            179:1    0  600M  0 part /boot/efi
├─mmcblk0p2            179:2    0    1G  0 part /boot
└─mmcblk0p3            179:3    0 28.2G  0 part 
  └─fedora_fedora-root 253:0    0 28.2G  0 lvm  /
zram0                  252:0    0  7.6G  0 disk [SWAP]
'

Finally our Physical & Logical Volume updated. Before reboot we needs to verify partitions using mount -a. If there is no more issue found then we can reboot our system.
