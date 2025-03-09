---
author: muju
title: "Disk Management on proxmoxVM"
description: ""
categories: ["Virtualization"]
image:
  path: /assets/vm/proxmox/harddisk.webp
  lqip: data:image/webp;base64,UklGRtYBAABXRUJQVlA4IMoBAADwCACdASooACgAPpFAm0klo6KhJzgMkLASCWoAzXf1EDrIPQASvrOSvWNqz5AzCBLJhor0LnqOMmfFzujIdIJt3Tyzwi0ir8HyknAt31AAD+2JiyHTWUpVQLWvcYPGsgxqt03bEGwS+On6xEZUciA//+uA/88Mm+ZBu//64DCbd74mUp+LJ6rgJT+dKKapiC8Q3Yk2o+kLgyRMejTePTbHKORD1mARjah/qwOr2UwEKr8YRLU3jDIR3TPehF2VkHyRvHNjE6hLejFy7pyXxTNS5IKqpMP+N19ecc208fIv/iAkE1PAin6bxTNn9M1giTVjyjL9d6WgNNggHwBoR76amx4Rxt6y7i/46cc0GL8GzWus5iyjyUmfxznAAgHX4JF+9affo7jTPpZ+PgsIpMKnV6pyXRPt0je+5loNTcylSZTAC7WZxf3eU8JkKeHw29BWVtN912wxqRy8+2Ydg6TgGeNPKqfrhCkZZ4N0IeXeEcUEL5muTmyvjIaLkei0hf7S+ZUeWEH89vFDOh++UBeQpRAeWWdnnwCNAq2dXd0R20tP4mRnT8YHFZ7B2+F6tmsKHBKgZG2NNXgTDZSNEuDX7+wyAAAAA==
#toc: false  #table of content
---

In this blog post, we will explore the process of increasing disk space for a Debian virtual machine (VM) running on Proxmox. This guide will detail the commands used and the output received during the resizing operation, providing a comprehensive overview for users looking to expand their VM's storage capacity.

# Step 1: Resize Disk in Proxmox

First, you need to increase the disk size from the Proxmox GUI. This can be done by selecting your VM, navigating to the "Hardware" tab, and adjusting the disk size accordingly. After resizing the disk in Proxmox, you can verify the new size by using the parted command.

```bash
root@pbs:~# parted                                                                                                                                                                            
GNU Parted 3.5                                                                                                                                                                                
Using /dev/sda                                                                                                                                                                                
Welcome to GNU Parted! Type 'help' to view a list of commands.                                                                                                                                
(parted)                                                                                                                                                                                      
(parted) print                                                                                                                                                                                
Warning: Not all of the space available to /dev/sda appears to be used, you can fix the GPT to use all of the space (an extra 23068672 blocks) or continue with the current setting?          
Fix/Ignore? f                                                                                                                                                                                 
Model: QEMU QEMU HARDDISK (scsi)                                                                                                                                                              
Disk /dev/sda: 22.5GB                                                                                                                                                                         
Sector size (logical/physical): 512B/512B                                                                                                                                                     
Partition Table: gpt                                                                                                                                                                          
Disk Flags:                                                                                                                                                                                   
                                                                                                                                                                                              
Number  Start   End     Size    File system  Name  Flags                                                                                                                                      
 1      17.4kB  1049kB  1031kB                     bios_grub                                                                                                                                  
 2      1049kB  538MB   537MB   fat32              boot, esp                                                                                                                                  
 3      538MB   10.7GB  10.2GB                     lvm                                                                                                                                        

```

# Step 2: Verify Disk Size
After fixing the partition table, print the partition information again:

```bash
(parted) resizepart 3 100%                                                 
(parted) print                                                             
Model: QEMU QEMU HARDDISK (scsi)
Disk /dev/sda: 22.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name  Flags
 1      17.4kB  1049kB  1031kB                     bios_grub
 2      1049kB  538MB   537MB   fat32              boot, esp
 3      538MB   22.5GB  22.0GB                     lvm

```

# Step 3: Resize Physical Volume

Next, resize the physical volume associated with your logical volume manager (LVM):

```bash
root@pbs:~# pvresize /dev/sda3
  Physical volume "/dev/sda3" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized
root@pbs:~# lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda            8:0    0   21G  0 disk 
├─sda1         8:1    0 1007K  0 part 
├─sda2         8:2    0  512M  0 part 
└─sda3         8:3    0 20.5G  0 part 
  ├─pbs-swap 252:0    0    1G  0 lvm  [SWAP]
  └─pbs-root 252:1    0  8.5G  0 lvm  /
sr0           11:0    1 1024M  0 rom  

root@pbs:~# df -h
Filesystem                             Size  Used Avail Use% Mounted on
udev                                   953M     0  953M   0% /dev
tmpfs                                  198M  772K  197M   1% /run
/dev/mapper/pbs-root                   8.3G  7.7G  125M  99% /
tmpfs                                  986M  200K  986M   1% /dev/shm
tmpfs                                  5.0M     0  5.0M   0% /run/lock
tmpfs                                  198M     0  198M   0% /run/user/0
```

# Step 4: Extend Logical Volume
Now that the physical volume is resized, extend your logical volume using:

```bash
root@pbs:~# lvextend -r -l +100%FREE /dev/mapper/pbs-root                                       
  Size of logical volume pbs/root changed from 8.49 GiB (2174 extents) to <19.50 GiB (4991 extents).
  Logical volume pbs/root successfully resized. 
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/mapper/pbs-root is mounted on /; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 3
The filesystem on /dev/mapper/pbs-root is now 5110784 (4k) blocks long.
```

# Step 5: Verify Filesystem Size
Finally, check if the filesystem reflects the new size:
```bash
root@pbs:~# df -h
Filesystem                             Size  Used Avail Use% Mounted on
udev                                   953M     0  953M   0% /dev
tmpfs                                  198M  792K  197M   1% /run
/dev/mapper/pbs-root                    20G  7.8G   11G  43% /
tmpfs                                  986M  200K  986M   1% /dev/shm
tmpfs                                  5.0M     0  5.0M   0% /run/lock
tmpfs                                  198M     0  198M   0% /run/user/0
```

