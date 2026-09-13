---
title: 'Corrupt & Repair a Filesystem'
---


# Overview
- Create a docker container with qemu installed.
- Create a qemu VM in the container.
- Create a filesystem and plug to the VM.
- Corrupt the filesystem.
- Check and repair the filesystem.


# Lab
## Create VM
- Setup lab: [**Alpine Single Node**](/html/lab/alpine_single_node.html).
- Install needed packages to the VM.
```sh
vm$ apk add lsblk e2fsprogs-extra util-linux file
```


## Create Filesystem
- Create a disk image.
```sh
container$ qemu-img create -f qcow2 disk.qcow2 2G
```


- Hotplug disk image to VM.
```sh
# Before hotplug
vm$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0    8G  0 disk
├─sda1   8:1    0  300M  0 part /boot
├─sda2   8:2    0  1.9G  0 part [SWAP]
└─sda3   8:3    0  5.8G  0 part /

# Hotplug
# Switch from VM console to QEMU monitor:   Ctrl + a c
(qemu) drive_add 0 file=disk.qcow2,media=disk,if=none,id=mydrive
(qemu) device_add virtio-blk-pci,drive=mydrive,id=mydevice

# After hotplug
# Switch from QEMU monitor to VM console:   Ctrl + a c
vm$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0    8G  0 disk
├─sda1   8:1    0  300M  0 part /boot
├─sda2   8:2    0  1.9G  0 part [SWAP]
└─sda3   8:3    0  5.8G  0 part /
vda    253:0    0    2G  0 disk
```


- Set partition table type.
```sh
vm$ fdisk /dev/vda

# create a new empty DOS partition table
(fdisk) o
(fdisk) w
```


- Create a new partition.
```sh
vm$ fdisk /dev/vda

# add a new partition
(fdisk) n
#   partition type: primary partition
#   partition number: (default)
#   first sector: (default)
#   last sector: +1G
(fdisk) w
```

```sh
vm$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0    8G  0 disk 
├─sda1   8:1    0  300M  0 part /boot
├─sda2   8:2    0  1.9G  0 part [SWAP]
└─sda3   8:3    0  5.8G  0 part /
vda    253:0    0    2G  0 disk 
└─vda1 253:1    0    1G  0 part
```


- Make filesystem.
```sh
vm$ mkfs.ext4 /dev/vda1
```

- Check.
```sh
vm$ fsck -fn /dev/vda1
```

- Mount.
```sh
vm$ mkdir /mnt/vda1
vm$ mount /dev/vda1 /mnt/vda1
```


# Corrupt & Repair Filesystem
Filesystems can be corrupted in many ways. This section will focus on the inodes.

## Orphaned File
The inode link count is the number of filenames or hard links that point to the inode. An inode with a zero link count is known as an orphaned file. The fsck considers an orphaned file as a incomplete deletion error.


- Create test files.
```sh
vm$ echo 'Hello A' > /mnt/vda1/file_a
vm$ echo 'Hello B' > /mnt/vda1/file_b
vm$ echo 'Hello C' > /mnt/vda1/file_c
```


- Unmount to debugfs.
```sh
vm$ umount /mnt/vda1
vm$ debugfs -w /dev/vda1
```


- Check file info.
```sh
(debugfs) ls -l
#   inode   filetype & mode       link_count      uid    gid      size    last modification   name
#           (octal)
      2     40755               (2)             0      0        4096    13-Sep-2026 10:04   .
      2     40755               (2)             0      0        4096    13-Sep-2026 10:04   ..
     11     100644              (1)             0      0        8       13-Sep-2026 10:04   file_a
     13     100644              (1)             0      0        8       13-Sep-2026 10:04   file_b
     14     100644              (1)             0      0        8       13-Sep-2026 10:04   file_c

# filetype & mode: 100644
#     - 10 : file type
#     -  0 : special permission
#     - 644: owner,group,other permission
```

```sh
(debugfs) show_inode_info file_a
Inode: 11   Type: regular    Mode:  0644   Flags: 0x80000
Generation: 3585892460    Version: 0x00000000:00000002
User:     0   Group:     0   Project:     0   Size: 8
File ACL: 0
Links: 1   Blockcount: 8
Fragment:  Address: 0    Number: 0    Size: 0
 ctime: 0x6aa67531:d12564e0 -- Sun Sep 13 10:04:33 2026
 atime: 0x6aa67531:d0e85bdc -- Sun Sep 13 10:04:33 2026
 mtime: 0x6aa67531:d12564e0 -- Sun Sep 13 10:04:33 2026
crtime: 0x6aa67531:d0e85bdc -- Sun Sep 13 10:04:33 2026
Size of extra inode fields: 32
Inode checksum: 0x806f8cfd
EXTENTS:
(0):33280
```


- Set link count to zero.
```sh
(debugfs) set_inode_field file_a links_count 0
```

```sh
(debugfs) show_inode_info file_a
Inode: 11   Type: regular    Mode:  0644   Flags: 0x80000
Generation: 3585892460    Version: 0x00000000:00000002
User:     0   Group:     0   Project:     0   Size: 8
File ACL: 0
Links: 0   Blockcount: 8
Fragment:  Address: 0    Number: 0    Size: 0
 ctime: 0x6aa67531:d12564e0 -- Sun Sep 13 10:04:33 2026
 atime: 0x6aa67531:d0e85bdc -- Sun Sep 13 10:04:33 2026
 mtime: 0x6aa67531:d12564e0 -- Sun Sep 13 10:04:33 2026
crtime: 0x6aa67531:d0e85bdc -- Sun Sep 13 10:04:33 2026
Size of extra inode fields: 32
Inode checksum: 0x806f8cfd
EXTENTS:
(0):33280

(debugfs) quit
```


- Check fsck.
```sh
vm$ fsck -fn /dev/vda1
Pass 1: Checking inodes, blocks, and sizes
Deleted inode 11 has zero dtime.  Fix? no

Pass 2: Checking directory structure
Entry 'file_a' in / (2) has deleted/unused inode 11.  Clear? no

Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
Block bitmap differences:  -33280
Fix? no

Inode bitmap differences:  -11
Fix? no

/dev/vda1: ********** WARNING: Filesystem still has errors **********
/dev/vda1: 14/65536 files (0.0% non-contiguous), 13018/262144 blocks
```

- Repair with fsck.
```sh
vm$ fsck -f /dev/vda1 
Pass 1: Checking inodes, blocks, and sizes
Deleted inode 11 has zero dtime.  Fix<y>? yes
Pass 2: Checking directory structure
Entry 'file_a' in / (2) has deleted/unused inode 11.  Clear<y>? yes
Pass 3: Checking directory connectivity
/lost+found not found.  Create<y>? yes
Pass 4: Checking reference counts
Pass 5: Checking group summary information
Block bitmap differences:  -33280
Fix<y>? yes
Free blocks count wrong for group #1 (32636, counted=32637).
Fix<y>? yes
Free blocks count wrong (249125, counted=249126).
Fix<y>? yes
Free inodes count wrong for group #0 (8177, counted=8178).
Fix<y>? yes
Free inodes count wrong (65521, counted=65522).
Fix<y>? yes

/dev/vda1: ***** FILE SYSTEM WAS MODIFIED *****
/dev/vda1: 14/65536 files (0.0% non-contiguous), 13018/262144 blocks
```


- Post check.
```sh
vm$ fsck -fn /dev/vda1 
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/vda1: 14/65536 files (0.0% non-contiguous), 13018/262144 blocks
```


- After repair with fsck, the orphaned file (file_a) is deleted.
```sh
vm$ mount /dev/vda1 /mnt/vda1

vm$ tree /mnt/vda1
/mnt/vda1
├── file_b
├── file_c
└── lost+found
```


# References
- [QEMU Disk HotPlug](https://wiki.ubuntu.com/QemuDiskHotplug).
- [QEMU Manual](https://www.qemu.org/docs/master/system/qemu-manpage.html).
- [Fun with fsck and debugfs](https://www.linux.com/training-tutorials/fun-e2fsck-and-debugfs).
- [The debugfs Manual](https://man7.org/linux/man-pages/man8/debugfs.8.html).
- [The inode Manual](https://man7.org/linux/man-pages/man7/inode.7.html).
- [Source code debugfs](https://github.com/tytso/e2fsprogs/blob/master/debugfs).