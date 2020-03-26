---
date: '2017-04-13 10:21:07'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- linux
title: Expand Disk In Linux
---

#### 1. If the disk you used is a iscsi/SAN disk

* Backup whole data on the old disk
* Change the disk size in storage size     
* Export the disk to the Linux OS again(usually reboot to recogonize the new size disk)
* After booting, use fdisk to see if the new size has been deteced by OS

        # fdisk -l

* If yes, then try to boot into rescue mode by installation media

        boot> linux rescue


* Try to check if there any filesystem error on current filesystem
* If no, try to increase the partition size by fdisk
* Then increase the filesystem by resize2fs
* Then fsck check filesystem to make sure all filesystem is ok after resize
 
#### 2. if the disk you used is a virtual disk in virtual machine, there are following ways

##### 2.1 Copy method
* Attache new disk to VM
* Partition it and format it
* Create a directory to mount the partition, e.g: There are two partitions in old disk:

        /dev/sda1 -> /boot
        /dev/sda2 –> /

    Then you had better follow the same organization
     
        #mkdir /newdisk 
        #mkdir /newdisk/boot
        #mount /dev/sdb1 /newdisk/boot
        #mount /dev/sdb2 /newdisk/

* Copy the files from old disk to new disk use either below commands
         
        #cp -ax / /newdisk 
        #cp -ax /boot/* /newdisk/boot
              
    However, you can use the one partition too
    !!! The -a option preserves the original system as much as possible. 
    !!! The -x option limits cp to a single file system; this is necessary to avoid copying the /new-disk and /proc directories.

        /dev/sdb1 -> /boot
        /dev/sdb2 –> /

* Umount all the partition in new disk then use fsck to check if there exist any filesytem error

        #fsck.ext3 -f /dev/sdb1 
        #fsck.ext3 -f /dev/sdb2

* Mount the /root on new disk again, and change the etc/fstab file to appropriate value

        #mount /dev/sdb2 /mnt/newdisk1 
        #cd /mnt/newdisk; vi etc/fstab

* Mount the /boot on new disk again, and chagne the boot/grub/grub.conf to ppropriate value

        #mount /dev/sdb1 /mnt/newdisk/boot 
        #cd /mnt/newdisk/boot; vi grub/grub.conf
    
* Shutdown the server, remove the old disk
* Try to boot with intallation media and go to rescue mode
                
        boot> linux rescue

* Install grub

        #chroot /mnt/sysimage 
        #cd /boot #grub-install

##### 2.2 Tar method(don't suggested, for some bugs may exist there)
    
* dd method(not suggested if your new disk have different geo parameters than old one)
* Attach a new large file-based file to guest VM as new disk, e.g: /dev/sdb
* Go into run level 1 to shut down the system daemons and preserve the state of the logs, and to prevent users from logging in

        #telinit 1

* Try to dd the old disk to new disk

        #dd if=/dev/<old disk> of=/dev/<new disk> bs=32256

    !!! Note: To get better dd performance, here set bs= sector size * sectors * track, you can adjust it based on your disk geo

* Shutdown the guest VM
* Remove the old disk configuration from guest VM
* Boot up with installation media
* Go to recure mode

        boot> linux rescue   
            
* Assume the rescue mode detects the installation media and mount it in /mnt/sysimage
* Chroot to /mnt/sysimage
 
        #cd /mnt/sysimage

* Reinstall the grub

        #cd /boot; grub-install <device name>     
            
* Bootup with gparted cd, and resize the partion
* Resize the filesystem

#### Reference
* [Hard-Disk-Upgrade](http://www.faqs.org/docs/Linux-mini/Hard-Disk-Upgrade.html)