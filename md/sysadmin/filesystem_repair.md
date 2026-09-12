---
title: 'Check and Repair a Filesystem'
---


# Overview


# Lab
## Create VM
- Setup an [Alpine Single Node](/html/lab/alpine_single_node.html).
- Start VM.
```sh
container$ ./start_vm.sh
```

- Install needed packages to VM.
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


- Set partition table.
```sh
vm$ fdisk /dev/vda

# create a new empty DOS partition table
(fdisk) o
(fdisk) w
```


- Create partition.
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

- Check filesystem.
```sh
vm$ fsck -fn /dev/vda1
```

- Mount.
```sh
vm$ mkdir /mnt/vda1
vm$ mount /dev/vda1 /mnt/vda1
```


# References
- [QEMU Disk HotPlug](https://wiki.ubuntu.com/QemuDiskHotplug).
- [QEMU Manual](https://www.qemu.org/docs/master/system/qemu-manpage.html).