---
title: Network File System (NFS)
subtitle: '(Lab Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- The lab runs in a Docker container with two QEMU VMs inside.
- VM1 acts as NFS server, which exports the NFS filesystem.
- VM2 acts as NFS client, which mount the NFS filesystem.

:::
::: {.column}

```go
       VM1                    VM2
   (NFS Server)            (NFS Client)
    192.0.0.10              192.0.0.20
       eth1                    eth1
        +                        +
        +------------------------+
```

:::
::::::::::::::


# Initial Setup
- Setup lab: [**Alpine 2 Nodes**](/html/lab/alpine_2n.html).


# Export & Mount
## Server
- Install packages.
```sh
vm1$ apk add nfs-utils rsyslog util-linux e2fsprogs-extra
```

- Start NFS service.
```sh
vm1$ rc-service nfs start
```

- Configure NFS directory.
```sh
vm1$ cat /etc/exports
# -------------------------------------------------- #
# Accept clients from 192.0.0.0/24
/srv/nfs 192.0.0.0/24(rw,sync,no_subtree_check)
# -------------------------------------------------- #

vm1$ exportfs -a

vm1$ exportfs -v
/srv/nfs        192.0.0.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,ro)
```


## Client
- Install packages.
```sh
vm2$ apk add nfs-utils rsyslog util-linux e2fsprogs-extra
```

- Mount.
```sh
vm2$ mkdir /mnt/nfs

vm2$ mount -t nfs 192.0.0.10:/srv/nfs /mnt/nfs

vm2$ df -h
192.0.0.10:/srv/nfs       5.6G    220.9M      5.1G   4% /mnt/nfs
```


# Startup
## Server
- Auto-start NFS on boot.
```sh
vm1$ rc-update add nfs
```


## Client
- Auto-mount on boot.
```sh
vm2$ cat /etc/fstab
192.0.0.10:/srv/nfs /mnt/nfs nfs4 rw,_netdev 0 0

vm2$ rc-update add nfsmount
```


# References
- [The /etc/exports file manual](https://man7.org/linux/man-pages/man5/exports.5.html).
- [The exportfs command manual](https://man7.org/linux/man-pages/man8/exportfs.8.html).
- [Alpine NFS setup](https://wiki.alpinelinux.org/wiki/Setting_up_an_NFS_server).