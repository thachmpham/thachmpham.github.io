---
title: 'Disk Input/Output Error'
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
vm$ apk add device-mapper lsblk dmesg e2fsprogs-extra perl
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

- Inject IO errors to the last one-third part of dm0; from sector 3145728, size of 1048576 sectors.
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

vm$ dmesg -w
Buffer I/O error on dev dm-0, logical block 393216, async page read
```


# IO Error on File Operations
- Reset the dm0 device, remove all error injection.
```sh
vm$ dmsetup reload dm0 --table '0 4194304 linear /dev/loop0 0'

vm$ dmsetup resume dm0

vm$ dmsetup table
dm0: 0 4194304 linear 7:0 0
```

- Make, mount filesystem.
```sh
vm$ mkdir /mnt/dm0

vm$ mkfs.ext4 /dev/dm-0

vm$ mount /dev/dm-0 /mnt/dm0
```

- Create test files.
```sh
vm$ perl -e 'print "A" x 512' > /mnt/dm0/file_a
vm$ perl -e 'print "B" x 512' > /mnt/dm0/file_b
vm$ perl -e 'print "C" x 512' > /mnt/dm0/file_c
```

- Size of a block on /dev/dm-0 is 4096 bytes. Total 524288 blocks.
```sh
vm$ dumpe2fs -h /dev/dm-0
Block count:    524288
Block size:     4096
```

- file_a starts at block 34827, which corresponds to byte offset 34827 * 4096 = 142651392, or sector 42651392 / 512 = 278616.
```sh
# find block
vm$ debugfs -R 'stat file_a' /dev/dm-0
EXTENTS:
(0):34827

# convert block to byte offset
vm$ echo '34827 * 4096' | bc
142651392

# to verify, dump data at this offset
vm$ xxd -s 142651392 -l 512 /dev/dm-0
0880b000: 4141 4141 4141 4141 4141 4141 4141 4141  AAAAAAAAAAAAAAAA
0880b010: 4141 4141 4141 4141 4141 4141 4141 4141  AAAAAAAAAAAAAAAA
...
0880b1e0: 4141 4141 4141 4141 4141 4141 4141 4141  AAAAAAAAAAAAAAAA
0880b1f0: 4141 4141 4141 4141 4141 4141 4141 4141  AAAAAAAAAAAAAAAA

# convert byte to 512-byte sector offset
vm$ echo '142651392 / 512' | bc
278616

# file_a
#   starts at sector 278616
#   size of 1 sector
```


- file_b starts at block 34827, which corresponds to byte offset 34827 * 4096 = 142651392, or sector 42651392 / 512 = 278616.
```sh
# find block
vm$ debugfs -R 'stat file_b' /dev/dm-0
EXTENTS:
(0):34828

# convert block to byte offset
vm$ echo '34828 * 4096' | bc
142655488

# to verify, dump data at this offset
vm$ xxd -s 142655488 -l 512 /dev/dm-0
0880c000: 4242 4242 4242 4242 4242 4242 4242 4242  BBBBBBBBBBBBBBBB
0880c010: 4242 4242 4242 4242 4242 4242 4242 4242  BBBBBBBBBBBBBBBB
...
0880c1e0: 4242 4242 4242 4242 4242 4242 4242 4242  BBBBBBBBBBBBBBBB
0880c1f0: 4242 4242 4242 4242 4242 4242 4242 4242  BBBBBBBBBBBBBBBB

# convert byte to 512-byte sector offset
vm$ echo '142655488 / 512' | bc
278624

# file_b
#   starts at sector 278624
#   size of 1 sector
```

- Inject IO errors.
```sh

```


dmsetup reload dm0 << EOF
0 274456 linear /dev/loop0 0
274456 10 error
274466 3919838 linear /dev/loop0 274466
EOF



# References
- [LINBIT Generate IO Errors](https://kb.linbit.com/generic-linux-tips/dmsetup-creating-virtual-blkdev-to-generate-io-errors/)
- [LINBIT Simulate Disk IO Errors](https://kb.linbit.com/generic-linux-tips/how-to-simulate-disk-io-error/)
- [GENTOO Device Mapper](https://wiki.gentoo.org/wiki/Device-mapper)
- [Kernel Device Mapper](https://docs.kernel.org/admin-guide/device-mapper/index.html)
- [Stackoverflow Find Sector of a File](https://stackoverflow.com/questions/3444844/finding-what-hard-drive-sectors-occupy-a-file)