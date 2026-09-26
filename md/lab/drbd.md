---
title: 'Distributed Replicated Block Device (DRBD)'
subtitle: '(Lab Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- Create 2 QEMU VMs: VM1, VM2.
- Connect VMs through a bridge: br0.
- Setup DRBD for the VMs.

:::
::: {.column}

```go
        VM1                 VM2
    192.0.0.10           192.0.0.20
       eth1                 eth1
        +                    +
        +--------------------+
                  +
                 br0
              192.0.0.254
               Container
```

:::
::::::::::::::


# Setup
- Setup lab: [Alpine 2 Nodes](/html/lab/alpine_2n.html).
- Copy needed files to the container.

```sh
host$ cd lab/drbd/setup-1
host$ find . -type f -exec docker cp {} apk:/ws \;
```

- Install packages.
```sh
cont$ ./drbd_install_packages.sh
```

- Create disk partitions.
```sh
host$ ./drbd_create_partitions.sh
```

- Setup drbd devices.
```sh
host$ ./drbd_setup_devices.sh
```

- Make filesystem and mount.
```sh
vm1$ mkfs.ext4 /dev/drbd0
vm1$ mkdir -p /mnt/drbd0
vm1$ mount /dev/drbd0 /mnt/drbd0
```


# Check

:::::::::::::: {.columns}
::: {.column}

- Check vm1.
```sh
vm1$ lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb         8:16   0    4G  0 disk 
└─sdb1      8:17   0    2G  0 part 
  └─drbd0 147:0    0    2G  0 disk /mnt/drbd0

vm2$ drbdadm status
drbd0 role:Primary
  disk:UpToDate
  peer role:Secondary
    replication:Established peer-disk:UpToDate
```

:::
::: {.column}

- Check vm2.
```sh
vm2$ lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb         8:16   0    8G  0 disk 
└─sdb1      8:17   0    2G  0 part 

vm2$ drbdadm status
drbd0 role:Secondary
  disk:UpToDate
  peer role:Primary
    replication:Established peer-disk:UpToDate
```


:::
::::::::::::::

# References
- [Alpine DRBD](https://wiki.alpinelinux.org/wiki/Disk_Replication_with_DRBD)