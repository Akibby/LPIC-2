# fstab Configuration
fstab = File System Table
fstab uses a mountpoint and a drive identifier to mount drives to the system
The command `lsblk -f` will show detailed information about the drives attached to the system
The UUID is generally the "best" identifier for a drive 
The fstab file can be found at `/etc/fstab`
The fstab file can be directly edited with any text editor
    `sudo vi /etc/fstab`
An entry will need multiple values
- The identifier 
    - Typically UUID
- A mount point 
    - for example `/mnt/backup/`
- The formatting of the drive
    - `ext4`/`vfat`/`swap`/etc.
    - using the wrong file system type could lead to corruption if recovery is attempted
- Options
    - `defaults` - just apply the default options to the file
    - `ro` - read only
    - `user` - can mount this drive without `sudo`
    - `nofail` - don't fail boot if this drive has an issue
    - `sw` - this is swap
- Dump 
    - this item is deprecated but still needed
    - 0 - don't backup
    - 1 - do backup
- Filesystem check value
    - 0 - don't check for errors at boot
    - 1 - critical drive for your system
    - 2 - a secondary drive that isn't critical to boot
To test if the fstab is working and valid the command `sudo mount -a` will attempt to apply the fstab file without a reboot
    If there is an error it will warn you

---
# Swap Partitions
Swap allows the hard drive to work as RAM
    This can work in a pinch but is **much** slower than RAM
The command `free -h` will show your RAM usage
    `swapon -s` will also show how much swap is being used
There are some "rules" for how much swap to have but none hold entirely true
Swap can be assigned either via a partition or just as a file on the disk
    HDDs can benefit from a partition 
    Swap files can be more easily managed
Setting up a swap partition
    The tool `fdisk` can be used to create a partition on a disk
        `sudo fdisk /dev/sdd`
        `p` will show information about the disk
        `n` will create a partition on the disk
        Once the partition is created the formatting type will need to be changed
        `t` will allow you to change the type
        `L` will list the available options
        Find the "Linux swap" option and use the number it is assigned
        `w` will save the changes and close `fdisk`
    `sudo mkswap /dev/sdd1` will enable the disk as swap
Setting up a swap file
    `sudo dd if=/dev/zero of=/var/swap bs=1M count=1024`
        This will create a file `/var/swap` that is made of 1024 1M blocks and write it to disk as all 0's
    `sudo chmod 600 /var/swap` will set proper permissions
    `sudo mkswap /var/swap` will tell the system that this is a swap file
In order to begin using these new swap files `swapon` is used
    `sudo swapon /dev/sdd1`
    `sudo swapon /var/swap`
    `free -h` should show the new storage available
    `swapon -s` will list all the swap in use
    **NOTE:** the swap partition will not persist until added to `fstab`
Swap can be disabled but if it was being used it will likely cause the program using it to crash 
    If swap is removed from `fstab` on reboot it will not be available
    `swapoff` can be used to turn off swap
Swap can be used in a sort of "RAID" configuration
    `swapon -s` will show the priority of the swap
    if this priority is set to an equal value for multiple entries it will be used in parallel
        This isn't really typically ever necessary
