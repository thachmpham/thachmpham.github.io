---
title: 'Corrupt a Filesystem'
---


# Overview
When an inode becomes invalid, the filesystem is considered as corrupted. 

Reproduce cases:

- Create an orphaned file.
- Create an invalid file mode.


# Setup Lab
## Initial Setup
- Setup lab: [**Alpine 1 Node**](/html/lab/alpine_1n.html).
- Install packages to the VM.
```sh
vm$ apk add lsblk e2fsprogs-extra util-linux file
```


## Create a Filesystem
- Create a disk image.
```sh
container$ qemu-img create -f qcow2 disk.qcow2 2G
```


- Add the disk image to the VM.
```sh
# Switch from VM console to QEMU monitor:   Ctrl + A C

# Plug disk to VM
(qemu) drive_add 0 file=disk.qcow2,media=disk,if=none,id=mydrive
(qemu) device_add virtio-blk-pci,drive=mydrive,id=mydevice

# Switch from QEMU monitor to VM console:   Ctrl + A C

# Check if disk added
vm$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda    253:0    0    2G  0 disk
```


- Create a partition.
```sh
vm$ fdisk /dev/vda
(fdisk) o   # create a new empty DOS partition table
(fdisk) w   # write changes

vm$ fdisk /dev/vda
(fdisk) n   # add a new partition
            # partition type: primary partition
            # partition number: (default)
            # first sector: (default)
            # last sector: +1G
(fdisk) w   # write changes

vm$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda    253:0    0    2G  0 disk
└─vda1 253:1    0    1G  0 part
```


- Make a filesystem.
```sh
vm$ mkfs -t ext4 /dev/vda1
vm$ fsck -fn /dev/vda1
```

- Mount the filesystem.
```sh
vm$ mkdir /mnt/vda1
vm$ mount /dev/vda1 /mnt/vda1
```


# Create an Orphaned File
The inode link count is the number of filenames or hard links that point to the inode. An inode with a zero link count is known as an orphaned file. The fsck considers an orphaned file as a incomplete deletion error.


- Create a test file.
```sh
vm$ echo 'Hello A' > /mnt/vda1/file_a
```


- Unmount the filesystem.
```sh
vm$ umount /mnt/vda1
```


- Check links count of the file.
```sh
vm$ debugfs -w /dev/vda1

(debugfs) ls -l
#   inode   filetype & mode     link_count      uid    gid      size    last modification   name
     13     100644              (1)             0      0        8       13-Sep-2026 10:04   file_a

(debugfs) show_inode_info file_a
Inode: 13   Type: regular    Mode:  0644   Flags: 0x80000
Links: 1
```


- Set link count to zero.
```sh
(debugfs) set_inode_field file_a links_count 0

(debugfs) show_inode_info file_a
Inode: 13   Type: regular    Mode:  0644   Flags: 0x80000
Links: 0
```


- Check fsck.
```sh
vm$ fsck -fn /dev/vda1
Pass 2: Checking directory structure
Entry 'file_a' in / (2) has deleted/unused inode 13.  Clear? no
/dev/vda1: WARNING: Filesystem still has errors
```


- Mount and list files.
```sh
vm$ mount /dev/vda1 /mnt/vda1

vm$ ls /mnt/vda1/
ls: /mnt/vda1/file_a: Data consistency error
lost+found
```


# Create an Invalid-Mode File
- Create a test file.
```sh
vm$ echo 'Hello B' > /mnt/vda1
```


- Unmount.
```sh
vm$ umount /mnt/vda1
```


- Check file mode.
```sh
vm$ debugfs -w /dev/vda1

(debugfs) ls -l
#    inode     filetype & mode     link_count      uid    gid      size    last modification   name
     14        100644              (1)             0      0        8       18-Sep-2026 15:08   file_b

# filetype & mode: 100644 (octal)
#     10: file type, 0: special permission, 644: owner,group,other permissions
```


- Set an invalid file mode.
```sh
# new mode: 020644 (oct) = 8612 (dec)
#           02: character device, 0: special permission, 644: owner,group,other permissions
(debugfs) set_inode_field file_b mode 8612

(debugfs) ls -l
#    inode     filetype & mode     link_count      uid    gid      size    last modification   name
     14        020644              (1)             0      0        8       18-Sep-2026 15:08   file_b
```


- Check fsck.
```sh
vm$ fsck -fn /dev/vda1
Pass 2: Checking directory structure
Inode 14 (/file_b) is an illegal character device.
/dev/vda1: WARNING: Filesystem still has errors
```

- Mount and list files.
```sh
vm$ mount /dev/vda1 /mnt/vda1/
vm$ ls -lh /mnt/vda1
total 20K    
crw-r--r--    1 root     root      243,  10 Sep 18 15:08 file_b
```


# References
- [QEMU Disk HotPlug](https://wiki.ubuntu.com/QemuDiskHotplug).
- [QEMU Manual](https://www.QEMU.org/docs/master/system/QEMU-manpage.html).
- [Fun with fsck & debugfs](https://www.linux.com/training-tutorials/fun-e2fsck-and-debugfs).
- [The debugfs Manual](https://man7.org/linux/man-pages/man8/debugfs.8.html).
- [The inode Manual](https://man7.org/linux/man-pages/man7/inode.7.html).
- [Source code debugfs](https://github.com/tytso/e2fsprogs/blob/master/debugfs).