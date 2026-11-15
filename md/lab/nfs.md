---
title: Network File System (NFS)
subtitle: '(Lab Setup Series)'
---


# Overview

A lab to practice NFS with 2 QEMU VMs.

- VM1 is NFS server that exports the filesystem.
- VM2 is NFS client that mounts the filesystem.


# Requirement
- Initialize lab with the instructions at [**Alpine 2 Nodes**](/html/lab/alpine_2n.html).
- Install packages.
```sh
vm1$ apk add nfs-utils rsyslog util-linux e2fsprogs-extra
vm2$ apk add nfs-utils rsyslog util-linux e2fsprogs-extra
```

# Manual Setup
## Export NFS on Server

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Start NFS service.
```sh
vm1$ rc-service nfs start
```

- Step 2: Create NFS directory.
```sh
vm1$ mkdir -p /srv/nfs
vm1$ chmod -R 777 /srv/nfs
```

:::
::: {.column width=50%}

- Step 3: Export NFS directory.
```sh
vm1$ vi /etc/exports
# -------------------------------------------------- #
# Accept clients from 192.0.0.0/24
/srv/nfs 192.0.0.0/24(rw,sync,no_subtree_check)
# -------------------------------------------------- #

vm1$ exportfs -a

vm1$ exportfs -v
/srv/nfs        192.0.0.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,ro)
```

:::
::::::::::::::


## Mount NFS on Client
- Mount the NFS filesystem.
```sh
vm2$ mkdir /mnt/nfs

vm2$ mount -t nfs 192.0.0.10:/srv/nfs /mnt/nfs

vm2$ df -h
192.0.0.10:/srv/nfs       5.6G    220.9M      5.1G   4% /mnt/nfs
```


# NFS Configuration at Boot
## Export NFS at Boot
- Start NFS at boot on server.
```sh
vm1$ rc-update add nfs
```


## Mount NFS on Boot

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Start nfsmount at boot on client.
```sh
vm2$ rc-update add nfsmount
```

:::
::: {.column width=50%}

- Step 2: Add nfs directory to fstab.
```sh
vm2$ cat /etc/fstab
192.0.0.10:/srv/nfs /mnt/nfs nfs4 rw,_netdev 0 0
```

:::
::::::::::::::


# Automation Setup

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Follow requirement section to initialize lab.

- Step 2: Copy needed file to container.
```sh
host$ find lab/nfs -type f -exec docker cp {} apk:/ws
```

:::
::: {.column width=50%}

- Step 3: Setup NFS.
```sh
host$ docker exec apk /ws/nfs_setup.sh
```

:::
::::::::::::::


# References
- [The /etc/exports file manual](https://man7.org/linux/man-pages/man5/exports.5.html).
- [The exportfs command manual](https://man7.org/linux/man-pages/man8/exportfs.8.html).
- [Alpine NFS setup](https://wiki.alpinelinux.org/wiki/Setting_up_an_NFS_server).