---
title: 'Distributed Replicated Block Device (DRBD)'
subtitle: '(Lab Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- Create 2 QEMU VMs: VM1, VM2
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
- Copy needed files to container.

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
vm1$ mkfs -t ext4 /dev/drbd0
vm1$ mkdir -p /mnt/drbd0
vm1$ mount -t ext4 /dev/drbd0 /mnt/drbd0
```


# References
- [Alpine DRBD](https://wiki.alpinelinux.org/wiki/Disk_Replication_with_DRBD)