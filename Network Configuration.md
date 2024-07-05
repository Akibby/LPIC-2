# Configuring Physical Adapters
`ip addr` will show the configuration of your network interfaces
`sudo dhclient -r` will release your current DHCP address
    `sudo dhclient` will grab a new IP via DHCP
`sudo systemctl restart network-manager` will restart your networking system on most modern devices
Static IPs can be assigned in a number of different ways
- RHEL
    - `/etc/sysconfig/network-scripts`
    - This will contain a folder for each interface for the device
    - There is a configuration file for each interface
- Debian
    - `/etc/network/interfaces`
- There can be other automatic services that automatically manage this
    - On Ubuntu there is a `netplan` service that will override configuration
Interface names hold meaning
- en - Ethernet Adapter
- wl - Wireless LAN
- ww - a cellular wireless card
- xxo - built directly onto the motherboard
- xxp - in a PCI slot
- xxs - in a "hotplug" slot (typically USB but can be PCI)
- The numbers at the end don't really have a meaning
