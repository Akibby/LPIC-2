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
The RAM disk version is typically tied to the kernel version
The RAM disk can be found at `/boot/` typically `initrd.img` (sometimes `initramfs.img` on RHEL)
    The RAM disk is actually zipped with 2 different protocols, CPIO for the beginning microcode of the file and gzip for the end
On Debian systems the command `mkinitramfs` can be used to modify the RAM disk (on RHEL `mkinitrd`)
Both Debian and RHEL will use `unmkinitramfs` can be used to extract the RAM disk file
    The extracted items should include a `early/` folder that contains the microcode 
    The `main/` folder was zipped with `gzip` and contains the rest of the RAM disk
The extracted items could be manually modified and recompressed but this is not ideal
