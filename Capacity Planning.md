# General
`top` shows a general overview of utilization
`htop` is a "pretty" version of `top` but isn't always available
`sar` needs installed but is a versatile tool

---
# Memory
`free` shows memory in bytes
	`free -h` gives more readable info
	buff/cache is used by the OS to cache
	`free -hs 2` will collect data every 2 seconds

`sar -r` will output memory utilization (importantly in one whole line at a time)
`vmstat` is another tool for showing memory
	`vmstat --unit M 2` will output the `vmstat` metric in MB every 2 seconds
## Swap
Using swap is bad because it is slow
When being used it may be worth considering upgrading RAM
`sar -S` will show swap information
`sar -W` will show I/O for swap

## Key Stats
Watch for blocks leaving RAM and moving to swap

---
# Disk
`sudo iotop` will give an overview of disk I/O similar to `top` but more specialized
	press `a` or launch with `sudo iotop -a` to use accumulate mode
		This will show what is using disk over time instead of at a moment in time
`lsof` will list open files
	`lsof -p 123` can be used to show all files open by a given process
	`lsof -c pname` can be used to show all files open by a given process by name (this is less reliable than using pid)
	Pay attention to where the data files are stored you may want to move the folder to a new disk
`iostat` will show an overview of all disks
	`iostat -x` will show even more metrics
`sar -b` will show storage data for all disks

---
# Network
`ntop`/`ntopng` will need installed but can do a lot
`sudo iftop` will show networking info
`sar -n {DEV/IP/TCP/ICMP}` will show information based on the keyword given
`mtr` an advanced `traceroute` 
	`mtr -z` will show BGP info
`ip addr` will show interface configuration
`ss` / `netstat`
`sudo lsof -iTCP -sTCP:ESTABLISHED` can show what ports are open and being used