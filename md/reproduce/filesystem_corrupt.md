---
title: 'Corrupt a Filesystem'
---


# Overview
A filesystem can be corrupted when an inode becomes invalid. We will reproduce the below issues:

- Orphaned file.
- Invalid file mode.
- Two files sharing the same block.


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
container$ QEMU-img create -f qcow2 disk.qcow2 2G
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
vm$ mkfs.ext4 /dev/vda1
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


- Show inode links count of the file.
```sh
vm$ debugfs -w /dev/vda1
(debugfs) ls -l
#   inode   filetype & mode       link_count      uid    gid      size    last modification   name
#           (octal)
     11     100644              (1)             0      0        8       13-Sep-2026 10:04   file_a

# filetype & mode: 100644
#     10 (file type), 0 (special permission), 644 (owner,group,other permissions)

(debugfs) show_inode_info file_a
Links: 1
```


- Set inode link count to zero.
```sh
(debugfs) set_inode_field file_a links_count 0
(debugfs) show_inode_info file_a
Links: 0
```


- Check fsck.
```sh
vm$ fsck -fn /dev/vda1
/dev/vda1: ********** WARNING: Filesystem still has errors **********
/dev/vda1: 14/65536 files (0.0% non-contiguous), 13018/262144 blocks
```


# References
- [QEMU Disk HotPlug](https://wiki.ubuntu.com/QemuDiskHotplug).
- [QEMU Manual](https://www.QEMU.org/docs/master/system/QEMU-manpage.html).
- [Fun with fsck and debugfs](https://www.linux.com/training-tutorials/fun-e2fsck-and-debugfs).
- [The debugfs Manual](https://man7.org/linux/man-pages/man8/debugfs.8.html).
- [The inode Manual](https://man7.org/linux/man-pages/man7/inode.7.html).
- [Source code debugfs](https://github.com/tytso/e2fsprogs/blob/master/debugfs).