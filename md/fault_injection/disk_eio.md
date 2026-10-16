---
title: 'Disk IO Error'
subtitle: '(Fault Injection Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column width=50%}

Simulate the disk IO error (EIO) with dm-error:

- Create a regular file.
- Create a loop device on top of the file.
- Create a dm-linear device on top of the loop device.
- Inject errors to an erea of the dm-linear device.
- Test read, write on the error erea.

:::
::: {.column width=50%}

| Term/Tool | Usage |
| :--- | :--- |
| Device Mapper (DM) | Map software devices to physical device. |
| Loop Device | Allow users to mount a regular file as a disk. |
| dmsetup | Manage DM devices. |
| losetup | Manage loop device. |
| dm-linear | Split, merge devices |
| dm-error  | Return I/O errors for any read, write to an erea. |
| sector  | In dmsetup, 1 sector = 512 bytes |

:::
::::::::::::::


# Setup Lab
- Setup: [**Alpine 1 Node**](/html/lab/alpine_1n.html).
- Install packages.
```sh
vm$ apk add device-mapper lsblk dmesg e2fsprogs-extra
```


# Create Devices
- Create a regular file, size of 2G.
```sh
vm$ dd if=/dev/zero of=file0.img bs=1M count=2048
```

- Create a loop device on top of the file.
```sh
# find a free loop device
vm$ losetup -f
/dev/loop0

# create loop device
vm$ losetup /dev/loop0 file0.img

# list loop devices
$ losetup -a
/dev/loop0: 0 file0.img

vm$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0    2G  0 loop
```

- Create a dm-linear device on top of the loop device.
```sh
# get size of loop device, unit: 512-byte sector
vm$ blockdev --getsz /dev/loop0
4194304

# create the dm-linear device
vm$ dmsetup create dm0 --table '0 4194304 linear /dev/loop0 0'

# show dm table
vm$ dmsetup table
dm0: 0 4194304 linear 7:0 0

vm$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0    2G  0 loop
└─dm0  253:0    0    2G  0 dm
```


# Inject IO Errors
- Show partition tables. The dm-linear device has 4194304 sectors. Size of a sector is 512 bytes.
```sh
vm$ fdisk -l
Disk /dev/dm-0: 2048 MB, 2147483648 bytes, 4194304 sectors
261 cylinders, 255 heads, 63 sectors/track
Units: sectors of 1 * 512 = 512 bytes
```

- Inject IO errors to the last one-third part of dm0, from sector 3145728, size of 1048576 sectors.
```sh
# inject IO errors
vm$ dmsetup reload dm0 << EOF
0 3145728 linear /dev/loop0 0
3145728 1048576 error
EOF
```

- Activate the setting.
```sh
# after injected, the setting still inactive
vm$ dmsetup table --inactive
dm0: 0 3145728 linear 7:0 0
dm0: 3145728 1048576 error

# resume to activate the setting
vm$ dmsetup resume dm0

# show dm table
vm$ dmsetup table
dm0: 0 3145728 linear 7:0 0
dm0: 3145728 1048576 error
```


# IO Error on Block Operations
- Read from the error area with dd.
```sh
vm$ dd if=/dev/dm-0 of=/dev/null bs=512 count=1 skip=3145728
dd: /dev/dm-0: I/O error

vm$ dmesg -wT
Buffer I/O error on dev dm-0, logical block 393216, async page read
```


# IO Error on File Operations
- Reset the dm0 device, remove all error injection.
```sh
vm$ dmsetup reload dm0 --table '0 4194304 linear /dev/loop0 0'

vm$ dmsetup resume dm0

vm$ dmsetup table dm0
0 4194304 linear 7:0 0
```

- Make filesystem and mount.
```sh
vm$ mkdir /mnt/dm0

vm$ mkfs.ext4 /dev/dm-0

vm$ mount /dev/dm-0 /mnt/dm0
```

- Create test files.
```sh
vm$ echo 'Hello A' > /mnt/dm0/file_a
vm$ echo 'Hello B' > /mnt/dm0/file_b
vm$ echo 'Hello C' > /mnt/dm0/file_c
```

- Get block size.
```sh
vm$ dumpe2fs -h /dev/dm-0
Block size:     4096

# size of each block on /dev/dm-0: 4096 bytes
```

- Find the sector of file_a.
```sh
# find the block where file_a locates
vm$ debugfs -R 'stat file_a' /dev/dm-0
Size: 8     # size of file_a:       8 bytes
EXTENTS:
(0):33280   # location of file_a:   block 33280

# convert block to byte offset
vm$ echo '33280 * 4096' | bc
136314880

# print data within the offset
vm$ xxd -s 136314880 -l 16 /dev/dm-0
08200000: 4865 6c6c 6f20 410a 0000 0000 0000 0000  Hello A.........

# convert byte to sector offset
vm$ echo '136314880 / 512' | bc
266240

# file_a
#   starts at sector 274504.
#   size of 8 bytes (within 1 sector).
```

- Find the sector of file_b.
```sh
# find the block where file_b locates
vm$ debugfs -R 'stat file_b' /dev/dm-0
Size: 8
EXTENTS:
(0):33792

# convert block to byte offset
vm$ echo '33792 * 4096' | bc
138412032

# print data within the offset
vm$ xxd -s 138412032 -l 16 /dev/dm-0
08400000: 4865 6c6c 6f20 420a 0000 0000 0000 0000  Hello B.........

# convert byte to sector offset
vm$ echo '138412032 / 512' | bc
270336

# file_b
#   starts at sector 270336.
#   size of 8 bytes (within 1 sector).
```

- Find the sector of file_c.
```sh
# find the block where file_c locates
vm$ debugfs -R 'stat file_c' /dev/dm-0
Size: 8
EXTENTS:
(0):33793

# convert block to sector offset
vm$ echo '33793 * 4096 / 512' | bc
270344

# file_c
#   starts at sector 270344.
#   size of 8 bytes (within 1 sector).
```


- Inject IO errors to the sector of file_b.
```sh
# layout
# sector 0      to sector 270336:   dm-linear
# sector 270336 to sector 270337:   dm-error
# sector 270337 to sector 4194304:  dm-linear

vm$ dmsetup reload dm0 << EOF
0 270336 linear /dev/loop0 0
270336 1 error
270337 3923967 linear /dev/loop0 270337
EOF

vm$ dmsetup resume dm0

vm$ dmsetup table dm0
0 270336 linear 7:0 0
270336 1 error 
270337 3923967 linear 7:0 270337
```

- Unmount and mount.
```sh
vm$ umount /mnt/dm0

vm$ mount /dev/dm-0 /mnt/dm0
```

- Read data from file_b, which injected error.
```sh
vm$ cat /mnt/dm0/file_b
cat: read error: I/O error
```

- Read data from file_a, file_c, which not injected error.
```sh
vm$ cat /mnt/dm0/file_a
Hello A

vm$ cat /mnt/dm0/file_c
Hello C
```

- Cleanup after test, remove error injection.
```sh
vm$ dmsetup reload dm0 --table '0 4194304 linear /dev/loop0 0'

vm$ dmsetup resume dm0

vm$ dmsetup table dm0
0 4194304 linear 7:0 0
```

- After remove error, able to read the files again.
```sh
vm$ cat /mnt/dm0/file_a
Hello A

vm$ cat /mnt/dm0/file_b
Hello B

vm$ cat /mnt/dm0/file_c
Hello C
```


# References
- [LINBIT Generate IO Errors](https://kb.linbit.com/generic-linux-tips/dmsetup-creating-virtual-blkdev-to-generate-io-errors/)
- [LINBIT Simulate Disk IO Errors](https://kb.linbit.com/generic-linux-tips/how-to-simulate-disk-io-error/)
- [GENTOO Device Mapper](https://wiki.gentoo.org/wiki/Device-mapper)
- [Kernel Device Mapper](https://docs.kernel.org/admin-guide/device-mapper/index.html)
- [Stackoverflow Find Sector of a File](https://stackoverflow.com/questions/3444844/finding-what-hard-drive-sectors-occupy-a-file)