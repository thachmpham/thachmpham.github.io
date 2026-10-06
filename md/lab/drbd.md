---
title: 'Distributed Replicated Block Device (DRBD)'
subtitle: '(Lab Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- This NFS lab is based on the [**Alpine 2 Nodes**](/html/lab/alpine_2n.html) lab.
- Used disks in VMs:
    - /dev/sda: Boot and filesystem.
    - /dev/sdb: For DRBD.
- The r0 DRBD resource replicates data between the two VMs.

:::
::: {.column}

```go
        VM1                 VM2
    192.0.0.10           192.0.0.20
       eth2                 eth2
        +                    +
        +--------------------+
                  +
                 br2
              192.0.0.254
               Container
```

:::
::::::::::::::


# Base Lab
- Setup lab: [Alpine 2 Nodes](/html/lab/alpine_2n.html).
- Install packages.
```sh
vm1$ apk add drbd-utils lsblk parted e2fsprogs-extra
vm2$ apk add drbd-utils lsblk parted e2fsprogs-extra
```


# Manual Setup
## Setup VM1
- Edit PATH to fix some minor issues.
```sh
vm1$ cat /root/.profile
export PATH=$PATH:/usr/lib/drbd

vm1$ cat /etc/init.d/drbd
export PATH=$PATH:/usr/lib/drbd
# ...
```

- Create the disk partition.
```sh
vm1$ parted -s /dev/sdb mklabel msdos
vm1$ parted -s /dev/sdb mkpart primary 0% 2GiB
```

- Configure DRBD resource.
```sh
vm1$ vi /etc/drbd.d/r0.res
# ---------------------------------------- #
resource r0 {
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

- Bring r0.
```sh
vm1$ drbdadm create-md r0
vm1$ drbdadm up r0
```

- Set as primary.
```sh
vm1$ drbdadm primary --force r0
```


## Setup VM2
- Edit PATH to fix some minor issues.
```sh
vm2$ cat /root/.profile
export PATH=$PATH:/usr/lib/drbd

vm2$ cat /etc/init.d/drbd
export PATH=$PATH:/usr/lib/drbd
# ...
```

- Create the disk partition.
```sh
vm2$ parted -s /dev/sdb mklabel msdos
vm2$ parted -s /dev/sdb mkpart primary 0% 2GiB
```

- Configure DRBD resource.
```sh
vm2$ vi /etc/drbd.d/r0.res
# ---------------------------------------- #
resource r0 {
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
vm2$ drbdadm create-md r0
vm2$ drbdadm up r0
```

- Set as secondary.
```sh
vm1$ drbdadm secondary r0
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
  └─drbd0 147:0    0    2G  0 disk

vm2$ drbdadm status
r0 role:Primary
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
r0 role:Secondary
  disk:UpToDate
  peer role:Primary
    replication:Established peer-disk:UpToDate
```

:::
::::::::::::::


# Automation Setup
To save time from manual setup, use the below steps to quickly setup the DRBD lab.

```sh
host$ find lab/drbd/setup-1 -type f -exec docker cp {} apk:/ws \;
host$ docker exec apk /ws/drbd/drbd_setup_vm1.sh
host$ docker exec apk /ws/drbd/drbd_setup_vm2.sh
```


# References
- [Alpine DRBD](https://wiki.alpinelinux.org/wiki/Disk_Replication_with_DRBD)