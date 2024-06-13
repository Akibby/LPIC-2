# Kernel Components

Kernel connects the hardware to the software and manages access
The kernel does not provide
- GUI
- bash
- service (apache, etc.)
- system services (systemd)
Does provide
- memory manager
- process manager
- hardware controller
- disk filesystem
The kernel can typically be found in `/boot`
	Name will be similar to `vmlinux`
	Sometimes modified to indicate the kind of compression used to zip the file
		`vmlinuz` means it is zipped by `gzip`
Monolithic vs Micro kernel
	Monolithic
		Everything part of the OS runs in kernel space
		In reality - it uses modules that can be dynamically loaded
		Will only ever grab and use modules that it actually needs
	Micro
		Kernel is as small as possible but things are run in user space
`initrd.img`
	initialization ram disk image
	GRUB will use this when booting to use modules that it needs before the kernel is online
	Things like storage drivers will be in here
Kernel modules can be found at `/lib/modules`
	`/lib/modules/initrd` contains modules used by `initrd.img`
	`/lib/modules/kernel` contains modules used by `vimlinux`/`vmlinuz` or whatever the kernel is being called
`/usr/src`
	Sometimes the kernel source code can be found here
	Official Linux kernel documentation can be found at https://www.kernel.org
	`linux-headers-` files can allow for checking that the kernel is running as expected

---
# Kernel Modules

`lsmod` will list all modules active on the system
`modinfo {mod}` will give info about whatever module is given
Kernel modules will be named `*.ko`
`demsg` will show module logs
`sudo insmod /lib/module/.../module.ko` can be used to install a specific module
`sudo rmmod *.ko` can be used to remove the specified module
`modprobe` is modern version of `insmod`/`rmmod` but is not on the LPIC-2 exam (according to instructor)
	`sudo modprobe -a ena` would add the `ena` module and `-r` would remove it
`lsusb` will show `usb` bus devices
`lspci` will show all `pci` devices
`lsdev` will show all devices
	this shows a lot of information 
	this is actually part of the `procinfo` package and may not always be installed
`/etc/udev` contains the `udev.conf` file
	`hwdb.d` this actually dictates the name but will be overwritten during an update. Use `rules.d` to modify names
	`rules.d` is a folder that contains files to rename devices
		refer to documentation for proper protocols on file names
		Ubuntu example
		File is `70-persistent-ipoib.rules`
			```SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="YOURMAC", ATTR{type}=="1", KERNEL=="ens33", NAME="eth0"```
		Follow this with a reboot to apply the change. Changes can sometimes be live pushed but not always.

---
# Kernel Compilation

Do you need to compile a custom kernel?
- Do you have special hardware that needs to be initialized before boot
- Do you need to exclude modules
- Is this for a Security Review
- Optimization
You will have to recompile a custom kernel for all future updates. This is a permanent commitment.
On Ubuntu to edit the kernel you need to modify `/etc/apt/sources.list` to allow it.
	uncomment the lines with `deb-src` to see the source code items
`sudo apt-get build-dep linux linux-image$(uname -r)` this will install the required build dependencies, Linux tools and the latest Linux image
These tools will allow you to make basic modifications to the kernel
Additional tools that may not be installed can be acquired with
	`sudo apt-get install build-essential lbncurses5-dev gcc libssl-dev grub2 bc bison flex libelf-dev fakeroot`
	Note that `libncurses5-dev` is optional
To pull the kernel source code from the distro run `sudo apt-get source linux-image-unsigned$(uname -r)`
To pull the official run `wget http://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.9.16.tar.xz` and modify the version and compression type in the command to whatever you want.
The kernel source uses the `.config` file for its config
    The file `/boot/config-5.4.0-67.generic` (or the equivalent one for your version) modifications to your kernel can be made.
    To make a new custom config file (if one doesn't already exist)
        `make defconfig` - for a default .config file
        `make xconfig` - launch a GUI tool to create a custom config
        `make menuconfig` - uses the `libncurses` package to give you a text menu to create the config
            `*` means it is built-in to the kernel and should be disabled if not needed
            `M` are modules and will only be loaded if needed
            Save when complete and you are done
To build the kernel you can just run `make`
    `make -j2` will allow you to instruct how many cores to use when building (2 in this case)
    `make -j2 deb-pkg` will build Debian Linux packages that can be used to install (wasn't done in course but mentioned)
    This build process will likely take 1+ hours
To start using your custom kernel
    If you have generated a `vmlinux` file you can copy it to `/boot` and tell GRUB to boot from it
        You can also symlink it but it still needs to be moved to `/boot`
    For Debian packages they can be installed via `apt`, `apt-get`, `dpkg` to install the local package
As long as you are using the same hardware across devices you can copy this kernel to other devices and set it up for use there without recompiling
The `initrd` may need to have its modules updated
The tool `dkms` can be used to build dynamic kernel modules
    may need to be installed with `apt install dkms`
    `dkms build -m ena -V 1.1` or whatever module to build the custom module
    `dkms install -m ena -V 1.1` to install the same module

---
# Monitoring the Kernel

the folder `/proc` 