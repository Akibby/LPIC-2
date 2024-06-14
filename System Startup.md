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
SysV init just runs a script that looks for a `rc.sysinit` or `rc.local` (this isn't standardized but this is generally the case).
SysV init does serial execution and if it hangs at one point all points afterwards will be delayed.
SysV init is no longer common and replaced by systemd generally but is around in some older servers.
