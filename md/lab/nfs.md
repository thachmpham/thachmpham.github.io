---
title: Network File System (NFS)
subtitle: '(Lab Series)'
---


# Overview
- This NFS lab is based on the [**Alpine 2 Nodes**](/html/lab/alpine_2n.html) lab.
- VM1 will be set up as the NFS server to export the filesystem.
- VM2 will be set up as the NFS client to mount the filesystem.


# Base Lab
- Setup lab: [**Alpine 2 Nodes**](/html/lab/alpine_2n.html).
- Install packages.
```sh
vm1$ apk add nfs-utils rsyslog util-linux e2fsprogs-extra
vm2$ apk add nfs-utils rsyslog util-linux e2fsprogs-extra
```


# Manual Setup
The step-by-step procedure to set up the NFS server and client.

## Export NFS on Server
- Start NFS service.
```sh
vm1$ rc-service nfs start
```

- Configure NFS directory.
```sh
vm1$ mkdir -p /srv/nfs
vm1$ chmod -R 777 /srv/nfs

vm1$ cat /etc/exports
# -------------------------------------------------- #
# Accept clients from 192.0.0.0/24
/srv/nfs 192.0.0.0/24(rw,sync,no_subtree_check)
# -------------------------------------------------- #

vm1$ exportfs -a

vm1$ exportfs -v
/srv/nfs        192.0.0.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,ro)
```


## Mount NFS on Client
- Mount the NFS filesystem.
```sh
vm2$ mkdir /mnt/nfs

vm2$ mount -t nfs 192.0.0.10:/srv/nfs /mnt/nfs

vm2$ df -h
192.0.0.10:/srv/nfs       5.6G    220.9M      5.1G   4% /mnt/nfs
```


# Boot Configuration
## Export NFS on Boot
- Add nfs to startup.
```sh
vm1$ rc-update add nfs
```


## Mount NFS on Boot
- Add nfsmount to startup.
```sh
vm2$ rc-update add nfsmount
```

- Add nfs directory to fstab.
```sh
vm2$ cat /etc/fstab
192.0.0.10:/srv/nfs /mnt/nfs nfs4 rw,_netdev 0 0
```


# Automation Setup
To save time from manual setup, use the below steps to quickly setup the NFS lab.

```sh
host$ find lab/nfs -type f -exec docker cp {} apk:/ws
host$ docker exec apk /ws/nfs_setup.sh
```


# References
- [The /etc/exports file manual](https://man7.org/linux/man-pages/man5/exports.5.html).
- [The exportfs command manual](https://man7.org/linux/man-pages/man8/exportfs.8.html).
- [Alpine NFS setup](https://wiki.alpinelinux.org/wiki/Setting_up_an_NFS_server).