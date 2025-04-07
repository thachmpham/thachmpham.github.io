---
title:  'Disk Manipulation With FDisk'
---

# 1. Create Partition
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


# 2. Expand Partition
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


# 3. Shrink Partition
Resize partition /dev/sda9 from 10G to 5G.

- Unmount.
```sh
  
$ umount /mnt/test
  
```

- Resize filesystem.
```sh
  
$ e2fsck -f /dev/sda9
$ resize2fs /dev/sda9 5G
  
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
> Last sector:          +5G
> Command:              w
  
```

- Check filesystem.
```sh
  
$ e2fsck -f /dev/sda9
  
```

- Mount.
```sh
  
$ mount /dev/sda9 /mnt/test

$ lsblk -l
NAME    SIZE    TYPE    MOUNTPOINTS
sda9    5G      part    /mnt/test
  
```


# 4. Mount Partition Permanently
Mount partition /dev/sda9 to directory /mnt/test permanently.

- Get filesystem of partition.
```sh
  
$ lsblk -f
NAME    FSTYPE
sda9    ext4
  
```

- Edit /etc/fstab.
```sh
  
/dev/sda9   /mnt/test   ext4    defaults    0   0
  
```

- Mount
```sh
  
$ mount -a
  
```


# 5. Mount /root To New Partition
Move /root from /dev/sda4 to /dev/sda8.

- Backup /root.
```sh
  
$ cp -r /root /root_old
  
```

- Mount new partition.
```sh
  
$ mkdir /mnt/sda8
$ mount /dev/sda8 /mnt/root
  
```

- Copy /root to new partition.
```sh
  
$ cp -r /root/* /mnt/root
  
```

- Edit /etc/fstab.
```sh
  
/dev/sda8   /root   ext3    defaults    0   0
  
```

- Reboot.
```sh
  
$ reboot -h now
  
```


# References
- https://man7.org/linux/man-pages/man8/fdisk.8.html
- https://man7.org/linux/man-pages/man8/mkfs.8.html
- https://man7.org/linux/man-pages/man8/lsblk.8.html