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


# Manual Setup
## Initial
- Setup lab: [Alpine 2 Nodes](/html/lab/alpine_2n.html).
- Install packages.
```sh
vm1$ apk add drbd-utils lsblk parted e2fsprogs-extra

vm2$ apk add drbd-utils lsblk parted e2fsprogs-extra
```

- Add drbd to path.
```sh
vm1$ vi /root/.profile
export PATH=$PATH:/usr/lib/drbd

vm2$ vi /root/.profile
export PATH=$PATH:/usr/lib/drbd
```


## Setup VM1
- Create the disk partition.
```sh
vm1$ parted -s /dev/sdb mklabel msdos

vm1$ parted -s /dev/sdb mkpart primary 0% 2GiB
```

- Configure DRBD resource.
```sh
vm1$ vi /etc/drbd.d/drbd0.res
# ---------------------------------------- #
resource drbd0 {
    device minor 0;
    disk /dev/sdb1;
    meta-disk internal;
    protocol C;

    on vm1 {
        address       192.0.0.10:7789;
    }

    on vm2 {
        address       192.0.0.20:7789;
    }
}
# ---------------------------------------- #
```

- Bring up the resource.
```sh
vm1$ drbdadm create-md drbd0
vm1$ drbdadm up drbd0
```

- Set as primary.
```sh
vm1$ drbdadm primary --force drbd0
```


## Setup VM2
- Create the disk partition.
```sh
vm2$ parted -s /dev/sdb mklabel msdos

vm2$ parted -s /dev/sdb mkpart primary 0% 2GiB
```

- Configure DRBD resource.
```sh
vm2$ vi /etc/drbd.d/drbd0.res
# ---------------------------------------- #
resource drbd0 {
    device minor 0;
    disk /dev/sdb1;
    meta-disk internal;
    protocol C;

    on vm1 {
        address       192.0.0.10:7789;
    }

    on vm2 {
        address       192.0.0.20:7789;
    }
}
# ---------------------------------------- #
```

- Bring up the resource.
```sh
vm2$ drbdadm create-md drbd0
vm2$ drbdadm up drbd0
```

- Set as secondary.
```sh
vm1$ drbdadm secondary drbd0
```


## Check Status

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
  └─drbd0 147:0    0    2G  1 disk

vm2$ drbdadm status
drbd0 role:Secondary
  disk:UpToDate
  peer role:Primary
    replication:Established peer-disk:UpToDate
```

:::
::::::::::::::


# Automation Setup
To quickly setup a DRBD lab:

- Setup lab: [Alpine 2 Nodes](/html/lab/alpine_2n.html).
- Copy needed files to the container.
```sh
host$ find lab/drbd/setup-1 -type f -exec docker cp {} apk:/ws \;
```

- Install packages to the VMs.
```sh
cont$ /ws/drbd_install_packages.sh
```

- Create disk partitions.
```sh
cont$ /ws/drbd_create_partitions.sh
```

- Setup DRBD resources.
```sh
cont$ /ws/drbd_up_resources.sh
```

# References
- [Alpine DRBD](https://wiki.alpinelinux.org/wiki/Disk_Replication_with_DRBD)