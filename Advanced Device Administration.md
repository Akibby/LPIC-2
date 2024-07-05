# Configuring RAID
Hardware RAID is better but software RAID can be useful
RAID Types
- 0 - Striping (good performance but no redundancy)
- 1 - Mirroring (decent performance poor efficiency because you lose half of your storage)
- 4 - Striping w/ Parity (all parity is on one disk)
- 5 - Striping w/ Parity (parity is split across all disks)
- 6 - Striping w/ 2 blocks of parity
- 10 - Multiple Mirrors and Striping is done across many mirrors
Most common RAID configurations are 1 and 5
RAID tools typically aren't installed by default
    Ubuntu `sudo apt install mdadm`
If a disk was previously used in a RAID array it needs to have the previous config removed
    `sudo mdadm --zero-superblock /dev/sdb /dev/sdc`
#### RAID 1
`sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc`
Note that that `/dev/md0` number will need to be iterated as needed
#### RAID 5
`sudo mdadm --create /dev/md1 --level=5 --raid-devices=3 /dev/sdd /dev/sde /dev/sdf`
#### Status
`cat /proc/mdstat` will show you the status of the RAID configuration being applied
Note that you can use the device while these devices are still being configured but you won't have full redundancy
To start using the disk
    `sudo mkfs.ext4 /dev/md0`
    `sudo mkfs.ext4 /dev/md1`
    `sudo mount /dev/md0 /mnt/raid1`
    `sudo mount /dev/md1 /mnt/raid5`
`sudo mdadm --detail --scan` will output the current RAID configuration
`sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf` will create a configuration file so the RAID configuration will automatically apply on boot
These devices will need to be included in `/etc/fstab` to mount at boot
    `/dev/md0 /mnt/raid1 ext4 defaults 0 0`
    `/dev/md1 /mnt/raid5 ext4 defaults 0 0`

---
# Supporting IDE and SATA Disks
It is extremely rare that these disks *need* to be configured
The disks can be customized for performance needs though
`sudo lshw` will show all hardware information
    `sudo lshw -class disk` will only show disk information
`hdparm` can be used to enable and configure disk features
    `sudo apt install hdparm`
`sudo hdparm -I /dev/sda` will show configuration info about the disk
`sudo apt install smartmontools` will allow you to check SMART reports to see if a drive failure may happen soon
    `sudo smartctl -i /dev/sda`
    `sudo smartctl -t short /dev/sda` test can be "short" or "long" and will run in the background
    `sudo smartctl -a /dev/sda` will show the results from the last test
    
---
# Supporting Solid State Disks
TRIM is a tool for SSDs that reduces the number of writes it has to do. This helps lengthen the life of the disk
TRIM support and status can be found in the output of `sudo hdparm -I /dev/sda`
`sudo fstrim -v /` will force TRIM to run and free up space (this shouldn't need to be run typically)
NVMe has a controller built in directly on the disk and because of this `hdparm` can't be used on it since it works on SATA/IDE
    NVMe doesn't typically support TRIM because of these changes but another tool on the disk should automatically manage it
    `sudo apt install nvme-cli` can be used to manage an NVMe
    `nvme --help`
    `sudo nvme smart-log /dev/nvme0n1`

---
# iSCSI and SAN Storage
#### Setting up a server as a iSCSI target
`sudo apt install tgt`
`sudo apt install --now tgt`
`sudo vi /etc/tgt/conf.d/tgt.conf`
```
<target iqn.hostname:storage>
  backing-store /dev/sdb
  initiator-address 10.0.222.50 #This will restrict who can access the iscsi
```
`sudo systemctl restart tgt`
note that a firewall may need to allow the traffic
    `sudo ss -natp` will show what port iSCSI is using
#### Setting up an iSCSI client
`sudo apt install open-iscsi`
`sudo iscsiadm -m discovery -t st -p 10.0.222.51` will check the specified IP for an iSCSI target
`sudo vi /etc/iscsi/initiatorname.iscsi`
add the following 
```
InitiatorName=iqn.hostname:storage
```
`sudo cd /etc/iscsi/nodes`
In here the iSCSI node and its `default` config can be found
    to automatically connect open `default` and modify `node.startup = automatic`
    The `openiscsi` and `iscsid` service will need to be restarted
        If there is an issue you may need to reboot the client, this is a software issue in iSCSI client
`sudo iscsiadm -m session -o show` will show you if there is an active iSCSI session

---
# Logical Volume Manager
`sudo apt install lvm2`
- Physical Volumes
`sudo pvcreate /dev/sdb /dev/sdc /dev/sdd` will create a physical volume for each 
`sudo pvdisplay` or `sudo pvs` will show info on the physical volumes
- Volume Group
`sudo vgcreate vg1 /dev/sdb /dev/sdc /dev/sdd` will create a volume group `vg1` with the listed volumes
`sudo vgdisplay` or `sudo vgs` will show information on the volume groups
- Logical Volume
`sudo lvcreate -L 300G vg1 -n website`
`sudo lvdisplay` or `sudo lvs` will show information on logical volumes

Once the logical volume is created it is treated like any other disk.
It will need to be formatted and mounted to be used
    Note that the volume group created in the example will be at `/etc/dev/vg1/website`
the LVM can be modified after creation to expand or shrink
    `vgextend` can be used to add more physical volumes to a volume group
    `lvresize` can be used to add more volume group space to the logical volume
        Note that in order to actually use the added space a tool based on what formatting the lv is in