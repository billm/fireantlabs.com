---
title: "Proxmox ZFS Drive Replacement"
date: 2021-05-29T19:04:44-04:00
year: "2021"
month: "2021/05"
toc: false
comments: false
images:
tags:
draft: false
---

Drive replacement is an inevitable task with computers and one that can be
nerve-wracking when you aren't well prepared for it. I recently had a drive
failure on the root filesystem of one of my Proxmox[^proxmox] hypervisors. In
my installation, Proxmox makes use of a two drive mirrored ZFS file system for
it's root partition. This post is as much to ensure the notes I took to recover
from the degraded volume are easy for me to find as to share the steps to
hopefully make someone elses life easier in the future.

<!--more-->

When I built my ZFS array, best practice was to use the `/dev/disk/by-id/ata*` 
prefix for assigning drives to the array. As `/dev/sdX` names will also be
referenced on occasion throughout this post, a reference mapping them is below.

```
# Serial numbers masked out below
# New disk
/dev/sda = /dev/disk/by-id/ata-TOSHIBA_HDWD130_SERIAL1
# Existing good disk
/dev/sdb = /dev/disk/by-id/ata-TOSHIBA_HDWD130_SERIAL2
```

To prepare the replacement disk we can use `sgdisk` and `pve-efiboot-tool` to copy known good
partition tables and properly initialize the drive.

```bash
# Backup the partition table from an identical good disk
sgdisk --backup=table /dev/disk/by-id/ata-TOSHIBA_HDWD130_SERIAL2  
# Restore the partition table on to the replacement disk
sgdisk --load-backup=table /dev/disk/by-id/ata-TOSHIBA_HDWD130_SERIAL1  

# Alternately this can be done in one command
# sgdisk /dev/\[source\] -R /dev/\[destination\]  
  
# Randomize GUIDs
sgdisk -G /dev/disk/by-id/ata-TOSHIBA_HDWD130_SERIAL1  
  
# Use the tools Proxmox provides to format the second partition and initialize
# it
pve-efiboot-tool format /dev/disk/by-id/ata-TOSHIBA_HDWD130_SERIAL1-part2 \
  --force  
pve-efiboot-tool init /dev/disk/by-id/ata-TOSHIBA_HDWD130_SERIAL1-part2  
```


If the bad disk is still in the system and recognized by the operating system
the new disk can be inserted as so to replace the bad disk:

```bash
sudo zpool replace -f rpool \
  /dev/disk/by-id/ata-TOSHIBA_HDWD130_SERIAL-DEAD-part3 \
  /dev/disk/by-id/ata-TOSHIBA_HDWD130_SERIAL1-part3
```

Otherwise, if the bad disk has been pulled from the system, or become
completely unrecognized by the operating system, the below command
can be used:

```bash
zpool attach rpool \
  /dev/disk/by-id/ata-TOSHIBA_HDWD130_SERIAL2-part3
  /dev/disk/by-id/ata-TOSHIBA_HDWD130_SERIAL1-part3
```

[^proxmox]: [https://www.proxmox.com/en/proxmox-ve](https://www.proxmox.com/en/proxmox-ve)