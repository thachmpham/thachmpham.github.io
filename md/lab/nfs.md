---
title: Network File System (NFS)
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- Create 2 QEMU VMs, Alpine OS.
- Setup VM1 as NFS server.
- Setup VM2 as NFS client.
- From client, mount a NFS directory hosted by server.

:::
::: {.column}

```go
  VM1 (NFS Server)        VM2 (NFS Client)
    192.0.0.10              192.0.0.20
       eth1                    eth1
        +                        +
        +------------------------+
```

:::
::::::::::::::


# Prerequisites
- Refer to this post to create VMs, [**Alpine 2 Nodes**](/html/lab/alpine_2n.html).


# Basic Setup 
## Server - Host NFS Filesystem
- Install packages.
```sh
vm1$ apk add nfs-utils rsyslog
```

- Start NFS service.
```sh
vm1$ rc-service nfs start
```

```sh
vm1$ rc-status
nfs [  started  ]
```

- Configure NFS directory.
```sh
vm1$ cat /etc/exports 
# -------------------------------------------------- #
# Accept clients from 192.0.0.0/24
/srv/nfs 192.0.0.0/24(rw,sync,no_subtree_check)
# -------------------------------------------------- #
```

```sh
vm1$ exportfs -a
```

```sh
vm1$ exportfs -v
/srv/nfs        192.0.0.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,ro)
```


## Client - Mount NFS Filesystem
- Install packages.
```sh
vm2$ apk add nfs-utils rsyslog
```

- Mount.
```sh
vm2$ mkdir /mnt/nfs
vm2$ mount -t nfs 192.0.0.10:/srv/nfs /mnt/nfs
```

```sh
vm2$ df -h
192.0.0.10:/srv/nfs       5.6G    220.9M      5.1G   4% /mnt/nfs
```

```sh
vm1$ showmount -a
All mount points on vm1:
192.0.0.10:/srv/nfs
192.0.0.20:/srv/nfs
```


# Startup Setup
## Server - Start NFS on Boot
- Auto-start NFS on boot.
```sh
vm1$ rc-update add nfs
```


## Client - Mount NFS on Boot
- Auto-mount on boot.
```sh
vm2$ cat /etc/fstab
# -------------------------------------------------- #
192.0.0.10:/srv/nfs /mnt/nfs nfs4 rw,_netdev 0 0
# -------------------------------------------------- #
```

```sh
vm2$ rc-update add nfsmount
```



# References
- [The /etc/exports file manual](https://man7.org/linux/man-pages/man5/exports.5.html).
- [The exportfs command manual](https://man7.org/linux/man-pages/man8/exportfs.8.html).
- [Alpine NFS setup](https://wiki.alpinelinux.org/wiki/Setting_up_an_NFS_server).