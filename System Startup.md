# Managing systemd
It is important to remember that not all Linux machines will use `systemd` to manage their processes
The initialization system being used can be found with the command `ps aux | head` and check for PID 1
    Note that the process running may be using an alias (ex: `/sbin/init`)
    To verify what this file actually is you would run `ls -la /sbin/init` and check if it is a symlink to another process (a symlink to `/lib/systemd/systemd` in the Ubuntu example)
The command `systemctl` will list all items that are being used by `systemd`
For the list of official items `/lib/systemd/system/` used by your operating system
Custom items are placed in `/etc/systemd/system/` so that they are not overwritten during an update
Targets determine what the system wants to be doing
    `/lib/systemd/system` contains the `*.target` files
    they are text files that can be easily output with `cat` and viewed
    The `default.target` is a symlink that is used during boot
    Targets can be modified or read with `systemctl-set` or `systemctl-get`
        Example: `sudo systemctl set-default multi-user.target` to change `default.target` to boot to a command line instead of GUI
Services can be configured to start at boot or just started to run
    `sudo systemctl start apache2` will make apache run right away
    `sudo systemctl enable apache2` will make apache run on startup
    `sudo systemctl enable --now apache2` will enable it and start it right away
Services can be customized
    `/lib/systemd/system/apache2.service` could be modified to wait on another service before starting with the `After` key or modify its start command with `ExecStart` or many other changes can be made
    This file will need to be placed in `/etc/systemd/system`

---
# Managing SysV init
SysV init just runs a script that looks for a `rc.sysinit` or `rc.local` (this isn't standardized but this 
is generally the case).
    in the course a centos6 VM has `rc.sysinit` and `rc.local` in `/etc/rc.d/`
SysV init does serial execution and if it hangs at one point all points afterwards will be delayed.
SysV init is no longer common and replaced by systemd generally but is around in some older servers.
The folders `rcX.d` X=0-6 each correlate to scripts you would want to run in different scenarios or "run levels"
- `rc0.d` - Scripts to run during shutdown
- `rc1.d` - depends 
- `rc2.d` - depends 
- `rc3.d` -  multiuser mode
- `rc4.d` - depends 
- `rc5.d` -  GUI mode
- `rc6.d` - Scripts to run during reboot
To check what the run levels all do check the file `/etc/inittab`
These scripts are run in alphabetical order
    There will be a `Xlocal` that can be modified, other files would be overwritten during updates
The command `who -r` will output the current run level
The run level can be modified
    The `inittab` file can have the default level modified and then on reboot the new run level should be used
    Run level can be changed without reboot though
        Etiquette is to check who is on the system with `who` and notify them
        `sudo init 3` will modify the run level to 3
        `tellinit` is an older command that would allow you to modify the run level on a timer
SysV init can be made to run a service at boot
- Can just be added to the `local` file for a given run level
- `chkconfig` can be used to turn a service on/off which determines if it starts at boot
    - `sudo chkconfig --list` will show the status of services across all run levels
    - `sudo chkconfig --level 35 httpd on` would enable `httpd` for run levels 3 and 5 only
    - This only modifies the status for future boots (doesn't apply right away)
- `service` can be used to check the `status` of a service and `start`/`stop` it
    - sometimes `service` is allowed to `restart` but this depends

---
# System Recovery with GRUB
GRUB config file typically found at `/boot/grub/grub.cfg`
    This file should not be directly edited or they will be overwritten
The folder `/etc/default/` is the where the main configuration file lives
    If you wanted to edit the GRUB timeout timer you would modify the `grub` file here
The folder `etc/grub.d/`is where the scripts to detect installed OS live
   This is where you would add a custom OS to the boot selection or modify one 
Modified config files need to be pushed to grub
    `sudo update-grub` should rebuild the GRUB
    Sometimes command is `update-grub2`
    The command `grub-mkconfig` is sometimes available and has some useful custom flags
GRUB can be reinstalled without harming the OS
If you cannot boot to GRUB at all there are 2 common approaches to regain access to the system
    Use a live CD (like Ubuntu installer) and boot to it from the BIOS. It will include its own GRUB
    Disconnect your drive and put it in a system that is working. Use the working GRUB install to fix things
       You should make sure to take note of a way to identify the drives
       `lsblk -f` can give you the UUID of the drive that is working
Recovering a drive by mounting it to a new system
    On the working system run `lsblk -f` and make note of the UUID of your working drive
    Once the new drive is connected it will need to be mounted
        Create a mountpoint somewhere (`sudo mkdir /mnt/sdb2` in the lesson)
        Mount the drive (`sudo mount /dev/sdb2 /mnt/sdb2`)
    You should now be able to navigate to the drive and see the existing file system
        Be careful about modifying things like this, you don't wanna break something
    The command `sudo grub-install --root-directory=/mnt/sdb2 /dev/sdb` will install GRUB on the drive and recreate the configuration
    You can now move the drive back to the other system and try to boot it again.
        NOTE: There may be other issues with the drive and the failing GRUB could indicate a more widespread issue (like a dying hard drive)

---
# Customizing the Initial RAM Disk
This typically shouldn't need to be modified
    If a piece of hardware needs to be accessible at boot and it isn't that would be a valid case
The RAM disk version is typically tied to the kernel version
The RAM disk can be found at `/boot/` typically `initrd.img` (sometimes `initramfs.img` on RHEL)
    The RAM disk is actually zipped with 2 different protocols, CPIO for the beginning microcode of the file and gzip for the end
On Debian systems the command `mkinitramfs` can be used to modify the RAM disk (on RHEL `mkinitrd`)
Both Debian and RHEL will use `unmkinitramfs` can be used to extract the RAM disk file
    The extracted items should include a `early/` folder that contains the microcode 
    The `main/` folder was zipped with `gzip` and contains the rest of the RAM disk
The extracted items could be manually modified and recompressed but this is not ideal
The modules used by the RAM Disk can be modified by editing `/etc/initramfs-tools/modules`
The command `sudo update-initramfs` can be used to regenerate the RAM disk
    the flag `sudo update-initramfs -u` will recreate the file in place
    the flag `sudo update-initramfs -u -b ~/` will recreate the file and put it an a specified directory
This new file still needs to be added to the GRUB config
    The menu entry for your OS should be copied to a custom location and the `initrd` should be modified for the entry
    Running `sudo update-grub` will rebuild GRUB and create the new boot option using the custom RAM disk

---
# systemd Mount Units
Why would you use Mount Units instead of fstab
    Every mount point can have its own config file
    You can set dependencies vs in fstab it just loads everything in the file
Typically you would load a data disk with mount units
Mount point - where the drive will be mounted
Mount unit - the file that the mount will be configured in
The file name of the mount unit should match the path
    If you want to mount to `/home/akib/backup` file would be named `home-akib-backup.mount`
The file will be written in `/etc/systemd/system/`
File is written like
```
[Unit]
Description=My backup drive
[Mount]
What=/dev/disk/by-uuid/youruuid
Where=/home/akib/backup
Type=ext4
Options=default
[Install]
WantedBy=multi-user.target
```
Before you can test you must run
    `sudo systemctl daemon-reload`
    `sudo systemctl start mnt-backups.mount`
To make it apply when you boot
    `sudo systemctl enable mnt-backups.mount`

---
# Supporting Btrfs
Btrfs has support for many different tools
- RAID
- LVM
Btrfs isn't being widely used yet but will likely become common in the near future
You may need to install the necessary tools to use btrfs
    Ubuntu: `sudo apt install btrfs-progs`
To initialize a filesystem
    `sudo mkfs.btrfs /dev/sdb`
    Note that you don't have to give a drive volume `sdb1` 
    You can but it is optional
    `sudo mount /dev/sdb/ /mnt/btrfs-demo`
The command `sudo btfs filesystem show` will show details for all btrfs file systems
#### RAID setup
    `sudo mkfs.btrfs -m raid1 -d raid1 /dev/sdc /dev/sdd`
        This will configure RAID1 for both the drive metadata (-m) and the disk data (-d)
#### Subvolumes
`sudo mount /dev/sdb /mnt/database`
`sudo btrfs subvolume create /mnt/database/db`
`sudo btrfs subvolume create /mnt/database/tl`
To check if there are subvolumes
    `sudo btrfs subvolume list -t /mnt/database` (-t does table output)
#### Snapshots
`sudo btrfs subvolume snapshot /mnt/database/db /mnt/database/snapshots/db20240705`
    For real world use you may set a CRON job to run a command like the one above
To access the snapshot you can just browse into the destination folder
The files in the snapshot are hard linked back to the original
Note that snapshots do not protect from hardware failures

---
# Working with Encrypted Storage
LUKS encryption can easily encrypt during install of a Linux system
To install and configure LUKS after install
    `lsblk -f` will let you verify if the existing drives are already encrypted
    a disk that has been previously used should be "shredded" after encrypting
        `sudo shred -v --iterations=3 /dev/sdeX`
        This will take several hours just to do 1 iteration
        It will rewrite the disk with 0s
    `sudo apt install cryptsetup-bin`
    `sudo cryptsetup luksFormat /dev/sde1`
        This will completely erase the disk (not shred though)
    `sudo cryptsetup luksOpen /dev/sde1 storage` will allow you to create an alias for the volume
    `sudo mkfs.ext4 /dev/mapper/storage` will format the volume as ext4
        It is important to use an alias because the pointer in the mapper can change easily
    `sudo mount /dev/mapper/storage /mnt/storage`
To mount these encrypted filesystems in the future `/etc/crypttab` will need to be modified
    You can just add the alias to automatically mount at boot (you will be prompted for a password for it)
        `storage /dev/sde1`
    If your boot drive is encrypted you could write the password into `/etc/crypttab`
        `storage /dev/sde1 password123`
    The device needs to be added to `/etc/fstab also`
        `/dev/mapper/storage /mnt/storage ext4 defaults 0 0`
If this is a shared computer additional keys will need to be generated for other users to encrypt/decrypt data
    `sudo cryptsetup luksAddKey /dev/sde1`
    `sudo cryptsetup luksRemoveKey /dev/sde1`