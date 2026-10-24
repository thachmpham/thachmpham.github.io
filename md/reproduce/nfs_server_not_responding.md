---
title: 'NFS Server Not Responding'
subtitle: '(Reproduce & Troubleshoot Series)'
---


# Overview
According to [NFS manual](https://linux.die.net/man/5/nfs), the NFS client mount a filesystem with the below options:

```sh
nfs_client$ mount -t nfs -o recovery_method,timeo=n,retrans=n server:path path
```

- recovery_method: The method for NFS client to recovers on requests time out.
    - hard: Retry indefinitely. Default.
    - soft: Retry a limited number of times.
- timeo: Time to wait before retry. Unit: decisecond (0.1s).
    - NFS over TCP: Default timeo: 600. After each retransmission, increases the timeout by up to the maximum of 600 seconds.
    - NFS over UDP: Default timeo: 11. After each retransmission, doubles the timeout, up to a maximum timeout length of 60 seconds.
- retrans: Number of retries before apply recovery method. Default: 3. The NFS client generates a "server not responding" message after retrans retries,

"NFS server not responding" occurs when the NFS client cannot complete a RPC function within the timeout. Common reasons:

- Firewall block: Firewall settings block the NFS ports. Requests cannot reach the server.
- Network errors: Packet drop, high latency. Messages cannot reach NFS peers in time.
- Disk IO bottleneck: The server experiences massive write loads, cannot read or write data fast enough to reply in time.
- Resource exhausted: The server handles too many requests or suffers from CPU/RAM starvation, unable to accept new requests.


# Lab
- Setup lab: [Network File System (NFS)](/html/lab/nfs.html).
- VM1 acts as NFS server.
- VM2 acts as NFS client.

:::::::::::::: {.columns}
::: {.column width=50%}

```sh
vm1$ exportfs -v
/srv/nfs 192.0.0.0/24(sync,wdelay,hide,no_subtree_check,\
sec=sys,rw,root_squash,no_all_squash)

vm1:~# ss -an | grep :2049
tcp   LISTEN 0      64                   0.0.0.0:2049          0.0.0.0:*
tcp   ESTAB  0      0                 192.0.0.10:2049       192.0.0.20:693
tcp   LISTEN 0      64                      [::]:2049             [::]:*
```

:::
::: {.column width=50%}

```sh
vm2$ mount -t nfs -o timeo=100,retrans=2 192.0.0.10:/srv/nfs /mnt/nfs

vm2$ mount -v
192.0.0.10:/srv/nfs on /mnt/nfs type nfs4 (rw,relatime,vers=4.2,\
rsize=131072,wsize=131072,namlen=255,\
hard,fatal_neterrors=none,proto=tcp,timeo=100,retrans=2,sec=sys,client)
```

:::
::::::::::::::


# NFS Blocked by Firewall
## Reproduce
- NFS Server: Install iptables.
```sh
vm1$ apk add iptables
```

- NFS Server: Add iptable rules to drop request to NFS port.
```sh
vm1$ iptables --append INPUT --source 192.0.0.20 -p tcp --dport 2049 -j DROP
vm1$ iptables --append INPUT --source 192.0.0.20 -p udp --dport 2049 -j DROP

vm1$ iptables -nvL --line-numbers
Chain INPUT (policy ACCEPT 435 packets, 35580 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1      163 10860 DROP       tcp  --  *      *       192.0.0.20           0.0.0.0/0            tcp dpt:2049
2        0     0 DROP       udp  --  *      *       192.0.0.20           0.0.0.0/0            udp dpt:2049
```

- NFS Client: Check dmesg.
```sh
vm2$ dmesg -wT
[Tue Oct  6 06:16:53 2026] nfs: server 192.0.0.10 not responding, still trying
[Tue Oct  6 06:19:15 2026] nfs: server 192.0.0.10 not responding, timed out
[Tue Oct  6 06:19:35 2026] nfs: server 192.0.0.10 not responding, timed out
```

## Cleanup
- NFS Server: Remove iptable rules.
```sh
vm1$ iptables --delete INPUT 2
vm1$ iptables --delete INPUT 1
```


# NFS Message Drops
## Reproduce
- NFS Server: Simulate packet loss.
```sh
vm1$ tc qdisc add dev eth2 root netem loss 100%

vm1$ tc qdisc show dev eth2
qdisc netem 8001: root refcnt 2 limit 1000 loss 100% seed 13627998538068838059
```

- NFS Client: Monitor dmesg.
```sh
vm2$ dmesg -wT
[Tue Oct  6 06:32:13 2026] nfs: server 192.0.0.10 not responding, still trying
[Tue Oct  6 06:33:48 2026] nfs: server 192.0.0.10 not responding, timed out
[Tue Oct  6 06:34:48 2026] nfs: server 192.0.0.10 not responding, timed out
```

## Cleanup
- NFS Server: Remove the injected delay.
```sh
vm1$ tc qdisc del dev eth2 root netem

vm1$ tc qdisc show dev eth2
qdisc pfifo_fast 0: root refcnt 2 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
```


# NFS Hang due to Disk
## Reproduce
- NFS Server: Install packages.
```sh
vm1$ apk add device-mapper
```

- NFS Server: Create a loop device.
```sh
# create a regular file, size of 2G
vm1$ dd if=/dev/zero of=file0.img bs=1M count=2048

# find a free loop device
vm1$ losetup -f
/dev/loop0

# create loop device
vm1$ losetup /dev/loop0 file0.img

# list blocks
vm$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0    2G  0 loop
```

- NFS Server: Create a dm-linear device on top of the loop device.
```sh
# get size of loop device, unit: 512-byte sector
vm1$ blockdev --getsz /dev/loop0
4194304

# create the dm-linear device
vm$ dmsetup create dm0 --table '0 4194304 linear /dev/loop0 0'

# show dm table
vm$ dmsetup table
dm0: 0 4194304 linear 7:0 0

# list blocks
vm$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0    2G  0 loop
└─dm0  253:0    0    2G  0 dm
```

- NFS Server: Make filesystem.
```sh
vm1$ mkfs.ext4 /dev/dm-0
```

- NFS Server: Mount.
```sh
vm1$ mkdir -p /mnt/dm0
vm1$ mount /dev/dm-0 /mnt/dm0

vm1$ chmod -R 777 /mnt/dm0
```

- NFS Server: Export NFS.
```sh
vm1$ vi /etc/exports
/mnt/dm0 192.0.0.0/24(rw,sync,no_subtree_check)

vm1$ exportfs -ar

vm1$ exportfs -v
/mnt/dm0        192.0.0.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```

- NFS Client: Mount NFS.
```sh
vm2$ mkdir /mnt/dm0

vm2$ mount -t nfs -o timeo=100,retrans=2 192.0.0.10:/mnt/dm0 /mnt/dm0
```

- NFS Server: Suspend the disk.
```sh
vm1$ dmsetup suspend /dev/dm-0
```

- NFS Client: Create a file in the NFS directory.
```sh
vm2$ touch /mnt/dm0/hello
```

- NFS Client: Monitor dmesg.
```sh
vm2$ dmesg -wT
[Thu Oct  8 23:51:25 2026] nfs: server 192.0.0.10 not responding, still trying
```


## Cleanup
- NFS Server: Resume the disk.
```sh
vm1$ dmsetup resume /dev/mapper/dm0
```


# References
- [NFS Manual](https://linux.die.net/man/5/nfs)
- [Emulate Disk Latency with dm-delay](https://blogs.oracle.com/linux/emulating-disk-latency-with-dm-delay)
- [Emulate a slow block device with dm-delay](https://enodev.fr/posts/emulate-a-slow-block-device-with-dm-delay.html)