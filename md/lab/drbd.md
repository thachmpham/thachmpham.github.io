---
title: 'Distributed Replicated Block Device (DRBD)'
subtitle: '(Lab Setup Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column width=50%}

A lab to practice DRBD with 2 QEMU VMs.

- The VMs connect to each other through the below bridge interfaces.
    - br1 (191.0.0.0/24): For ssh.
    - br2 (192.0.0.0/24): For DRBD.
- Each VMs has the below disks.
    - /dev/sda: Boot and filesystem.
    - /dev/sdb: For DRBD.
- The r0 DRBD resource replicates data between the VMs.

:::
::: {.column width=50%}

The VMs run within a docker container which acts as router, forwards traffics between VMs.
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


# Requirement
- Initialize lab with the instructions at [Alpine 2 Nodes](/html/lab/alpine_2n.html).
- Install packages.
```sh
vm1$ apk add drbd-utils lsblk parted e2fsprogs-extra
vm2$ apk add drbd-utils lsblk parted e2fsprogs-extra
```


# Manual Setup
## Setup VM1

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Create the disk partition.
```sh
vm1$ parted -s /dev/sdb mklabel msdos
vm1$ parted -s /dev/sdb mkpart primary 0% 2GiB
```

- Step 2: Configure DRBD resource.
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

:::
::: {.column width=50%}

- Step 3: Edit PATH.
```sh
vm1$ vi /root/.profile
export PATH=$PATH:/usr/lib/drbd

vm1$ vi /etc/init.d/drbd
export PATH=$PATH:/usr/lib/drbd
```

- Step 4: Bring up r0.
```sh
vm1$ drbdadm create-md r0
vm1$ drbdadm up r0
```

- Step 5: Set vm1 as primary.
```sh
vm1$ drbdadm primary --force r0
```

:::
::::::::::::::


## Setup VM2

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Create the disk partition.
```sh
vm2$ parted -s /dev/sdb mklabel msdos
vm2$ parted -s /dev/sdb mkpart primary 0% 2GiB
```

- Step 2: Configure DRBD resource.
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

:::
::: {.column width=50%}

- Step 3: Edit PATH.
```sh
vm2$ vi /root/.profile
export PATH=$PATH:/usr/lib/drbd

vm2$ vi /etc/init.d/drbd
export PATH=$PATH:/usr/lib/drbd
```

- Step 4: Bring up r0.
```sh
vm2$ drbdadm create-md r0
vm2$ drbdadm up r0
```

- Step 5: Set vm1 as secondary.
```sh
vm1$ drbdadm secondary r0
```

:::
::::::::::::::


## Check DRBD Status

:::::::::::::: {.columns}
::: {.column}

- Check drbd on vm1.
```sh
vm2$ drbdadm status
r0 role:Primary
  disk:UpToDate
  peer role:Secondary
    replication:Established peer-disk:UpToDate

vm1$ lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb         8:16   0    4G  0 disk
└─sdb1      8:17   0    2G  0 part
  └─drbd0 147:0    0    2G  0 disk
```

:::
::: {.column}

- Check drbd on vm2.
```sh
vm2$ drbdadm status
r0 role:Secondary
  disk:UpToDate
  peer role:Primary
    replication:Established peer-disk:UpToDate

vm2$ lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb         8:16   0    8G  0 disk
└─sdb1      8:17   0    2G  0 part
  └─drbd0 147:0    0    2G  1 disk
```

:::
::::::::::::::


# Automation Setup

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Follow requirement section to initialize lab.

- Step 2: Copy needed file to container.
```sh
host$ find lab/drbd/setup-1 -type f -exec docker cp {} apk:/ws \;
```

:::
::: {.column width=50%}

- Step 3: Setup VMs.
```sh
host$ docker exec apk /ws/drbd_setup_vm1.sh
host$ docker exec apk /ws/drbd_setup_vm2.sh
```

- Step 4: Check DRBD status on the VMs.

:::
::::::::::::::


# References
- [Alpine DRBD](https://wiki.alpinelinux.org/wiki/Disk_Replication_with_DRBD)