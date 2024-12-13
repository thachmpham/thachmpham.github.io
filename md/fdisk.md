---
title:  'Disk Manipulation'
---


## 1. lsblk
- Find unpartitioned disk space.
```sh
  
$ lsblk -l
NAME    SIZE    TYPE    MOUNTPOINTS
sda     50G     disk    
sda1    9.6G    part    /
sda2    1K      part
sda3    1K      part
sda5    3.5G    part    /var
sda6    976M    part
sda7    685M    part    /tmp
sda8    20G     part    /home

# free  = sda - sda1 - sda2 - sda3 - sda5 - sda6 - sda7 - sda8
#       = 50  - 9.6                - 3.5  - 0.97 - 0.68 - 20
#       = 15.25 (G)
  
```


## 2. fdisk
### 2.1. Create Partition
Create a 5G partition on /dev/sda9.

- Create partition.
```sh
  
$ fdisk /dev/sda
> Command:          n
> First sector:     default
> Last sector:      +5G
> Command:          w

$ lsblk -l
NAME    SIZE    TYPE    MOUNTPOINTS
sda9    5G      part
  
```

- Make filesystem.
```sh
  
$ mkfs.ext4 /dev/sda9
  
```

- Mount.
```sh
  
$ mkdir /mnt/test
$ mount /dev/sda9 /mnt/test

$ lsblk -l
NAME    SIZE    TYPE    MOUNTPOINTS
sda9    5G      part    /mnt/test
    
```


### 2.3. Resize Partition
Resize partition /dev/sda9 from 5G to 10G.

- Unmount.
```sh
  
$ umount /mnt/test
  
```

- Delete partition /dev/sda9.
```sh
   
$ fdisk /dev/sda
> Command:              d
> Partition number:     9
> Command:              w

```

- Re-create partition /dev/sda9.
```sh
  
$ fdisk /dev/sda
> Command:              n
> First sector:         default
> Last sector:          +10G
> Command:              w
  
```

- Resize filesystem.
```sh
  
$ e2fsck -f /dev/sda9
$ resize2fs /dev/sda9 10G
  
```

- Mount.
```sh
  
$ mount /dev/sda9 /mnt/test

$ lsblk -l
NAME    SIZE    TYPE    MOUNTPOINTS
sda9    10G      part    /mnt/test
  
```

# References
- https://man7.org/linux/man-pages/man8/fdisk.8.html
- https://man7.org/linux/man-pages/man8/mkfs.8.html
- https://man7.org/linux/man-pages/man8/lsblk.8.html