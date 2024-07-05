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
