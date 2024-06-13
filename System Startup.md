# Managing systemd
It is important to remember that not all Linux machines will use `systemd` to manage their processes
The initialization system being used can be found with the command `ps aux | head` and check for PID 1
    Note that the process running may be using an alias (ex: `/sbin/init`)
    To verify what this file actually is you would run `ls -la /sbin/init` and check if it is a symlink to another process (a symlink to `/lib/systemd/systemd` in the Ubuntu example)
The command `systemctl` will list all items that are being used by `systemd`
For the list of official items `/lib/systemd/system/` used by your operating system
Custom items are placed in `/etc/systemd/system/` so that they are not overwritten during an update