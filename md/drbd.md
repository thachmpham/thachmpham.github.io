---
title: Distributed Replicated Block Device
---

# 1. Introduction
Distributed Replicated Block Device (DRBD) is a tool for data replication at the block level between servers.  

`drbd-utils` is the package for managing and configuring DRBD. The package provides the below commands:

- `drbdadm`: Manage resources (configure, start/stop, promote/demote).
- `drbdsetup`: Low-level control for debugging/custom setups.
- `drbdmeta`: Manage DRBD metadata (create, inspect, remove).
- `drbdmon`: Monitor real-time states and sync progress.


# 2. Lab
Set up DRBD to replicate a partition between two hosts:

- Install drbd-utils on node 1 and node 2.
- Modify /etc/drbd.conf on both nodes.
- Start drbd service.


## 2.1. Install DRBD
- Install DRBD in two nodes.
```sh
  
node1$ apt install drbd-utils  
node2$ apt install drbd-utils  
  
```


## 2.2. Configuration File

- Edit files /etc/drbd.conf on node 1 and node 2 as below.
```c++
  
global { usage-count no; }
common { syncer { rate 100M; } }

resource r0 {
    protocol C;
    startup {
        wfc-timeout 15;
        degr-wfc-timeout 60;
    }
    net {
        cram-hmac-alg sha1;
        shared-secret "secret";
    }
    on node1 {
        device /dev/drbd0;
        disk /dev/sda9;
        address 192.168.122.122:7788;
        meta-disk internal;
    }
    on node2 {
        device /dev/drbd0;
        disk /dev/sda9;
        address 192.168.122.28:7788;
        meta-disk internal;
    }
}
  
```

<pre class="mermaid">
flowchart TD
    node1_drbd0["/dev/drbd0"]
    node2_drbd0["/dev/drbd0"]

    subgraph node1
        node1_sda9["/dev/sda9"]
        node1_sda9 <---> node1_drbd0
    end

    subgraph node2
        node2_sda9["/dev/sda9"]
        node2_sda9 <---> node2_drbd0
    end

    node1_drbd0 <---> node2_drbd0
</pre>

- `/dev/sda9` is the existing partition.
- `/dev/drbd0` is drbd partition, no need to be pre-existing, it will be created automatically by drbd service.
- `/dev/sda9` serves as the underlying storage for `/dev/drbd0`.


## 2.3. Meta-data
- Create drbd meta-data.
```sh
  
node1$ drbdadm create-md r0
node2$ drbdadm create-md r0
  
```

## 2.4. Start DRBD
- Start drbd on node 1.
```sh
  
node1$ systemctl start drbd
  
```

```sh
  
# syslog
node1 systemd:  Starting drbd.service
node1 drbd:     [
node1 drbd:         create res:     r0
node1 drbd:         prepare disk:   r0
node1 kernel:           block drbd0:    disk( Diskless  ->  Attaching )
node1 kernel:           block drbd0:    disk( Attaching ->  Inconsistent )
node1 drbd:         adjust disk:    r0
node1 drbd:         adjust net:     r0
node1 kernel:           drbd r0:        conn( StandAlone    ->  Unconnected )
node1 drbd:     ]
node1 kernel:           drbd r0:        conn( Unconnected   ->  WFConnection )
node1 systemd:  Finished drdb.service
  
```

- Start drbd on node 2.
```sh
  
node2$ systemctl start drbd
  
```

```sh
  
# syslog
# node2 starts service
node2 systemd:  Starting drbd.service
node2 drbd:     [
node2 drbd:         create res:     r0
node2 drbd:         prepare disk:   r0
node2 kernel:           block drbd0:    disk( Diskless  ->  Attaching )
node2 kernel:           block drbd0:    disk( Attaching ->  Inconsistent )
node2 drbd:         adjust disk:    r0
node2 drbd:         adjust net:     r0
node2 kernel:           drbd r0:        conn( StandAlone    ->  Unconnected )
node2 drbd:     ]
node2 kernel:           drbd r0:        conn( Unconnected   ->  WFConnection )

# node2 handshakes node1
node2 kernel:   drbd r0:    Handshake sucessfull
node2 kernel:   drbd r0:    conn( WFConnection  ->  WFReportParams )
node2 kernel:   block r0:   peer( Unknown -> Secondary ), conn( WFReportParams -> Connected ), pdsk( DUnknown -> Inconsistent )
node2 systemds: Finish drbd.service

# node1 handshakes node2
node1 kernel:   drbd r0:    Handshake sucessfull
node1 kernel:   drbd r0:    conn( WFConnection  ->  WFReportParams )
node1 kernel:   block r0:   peer( Unknown -> Secondary ), conn( WFReportParams -> Connected ), pdsk( DUnknown -> Inconsistent )
node1 systemds: Finish drbd.service
  
```


## 2.5. Promote Primary
- Promote node 1 to primary and overwrite state of node 2.
```sh
  
node1$ drbdadm -- --overwrite-date-of-peer primary all
  
```

```sh
  
# syslog
# node1 becomes sync source
node1 kernel:   block drbd0:    role( Secondary  ->  Primary )   disk( Inconsistent    ->  UpToDate )
node1 kernel:   block drbd0:    conn( Connected  ->  WFBitMapS )
node1 kernel:   block drbd0:    conn( WFBitMapS  ->  SyncSource )

# node2 becomes sync target
node2 kernel:   block drbd0:    peer( Secondary  ->  Primary )   disk( Inconsistent    ->  UpToDate )
node2 kernel:   block drbd0:    conn( Connected  ->  WFBitMapT )
node2 kernel:   block drbd0:    conn( WFBitMapT  ->  WFSyncUUID )
node2 kernel:   block drbd0:    conn( WFSyncUUID ->  SyncTarget )

# sync done.
node1 kernel:   block drbd0:    conn( SyncSource  ->  Connected )   pdsk (Inconsistent  ->  UpToDate )
node2 kernel:   block drbd0:    conn( SyncTarget  ->  Connected )   pdsk (Inconsistent  ->  UpToDate )
  
```

- Monitor sync progress.
```sh
  
$ watch -n1 cat /proc/drbd
  
```

- Check drbd status.
```sh
  
$ drbdadm status
  
```


## 2.6. Make Filesystem
- After sync finished, drbd partition /dev/drbd0 will be created automatically by drbd service.
```sh
  
node1$ lsblk -f
NAME        FSTYPE      MOUNTPOINTS
sda
├─ sda8     ext4        /home
└─ sda9     drbd
   └─ drbd0
  
```

```sh
  
node2$ lsblk -f
NAME        FSTYPE      MOUNTPOINTS
sda
├─ sda8     ext4        /home
└─ sda9     drbd
   └─ drbd0
  
```

- Make filesystem for drbd partition.
```sh
  
$ mkfs.ext3 /dev/drbd0
  
```


## 2.8. Mount
- On node1, mount drbd partition.
```sh
  
node1$ mkdir /mnt/test
node1$ mount /dev/drbd0 /mnt/test
  
```

*Notes: the drbd partition can only be mounted on the primary node.*


# 3. Fault Tolerance
## 3.1. Simulate Fault on Node 1
- Create a file on node 1.  
```sh
  
node1$ touch /mnt/test/hello
  
```

- Stop drbd on node 1.
```sh
  
node1$ systemctl stop drbd
  
```

```sh
  
# syslog
# node1 stops drbd
node1 systemd:  Stopping drbd.service
node1 kernel:   EXT4-fs (drbd0): unmounting filesystem
node1 drbd:     Stopping all DRBD resources
node1 kernel:   block drbd0:    role( Primary   ->  Secondary )
node1 kernel:   drbd r0:        peer( Secondary ->  Unknown ), conn(Connected   ->  Disconnecting ) pdsk( UpToDate   ->  DUnknown )
node1 kernel:   drbd r0:        conn( Disconnecting ->  StandAlone )
node1 kernel:   drbd r0:        disk( UpToDate  ->  Failed )
node1 kernel:   drbd r0:        disk( Failed    ->  Diskless )
node1 systemds: Stopped drbd.service

# node2 updates state
node2 kernel:   block drbd0:    peer( Primary   ->  Secondary)
node2 kernel:   drbd r0:        peer( Secondary ->  Unknown ) conn( Connected   ->  TearDown ) pdsk( UpToDate   ->  DUnknown )
node2 kernel:   drbd r0:        conn( TearDown  ->  Unconnected )
node2 kernel:   drbd r0:        conn( Unconnected  ->  WFConnection )
  
```

## 3.2. Recover on Node 2
- Promote node 2 to primary.
```sh
  
node2$ drbdadm primary r0
  
```

```sh
  
# syslog
node2 kernel: block drbd0: role( Secondary  ->  Primary )  
  
```

- Mount drbd partition.
```sh
  
node2$ mount /dev/drbd0 /mnt/test
  
```

- Check if the file can be recoverd on node 2.
```sh
  
node2$ ls /mnt/test
hello
  
```


# References
- [manpages.ubuntu.com/manpages/xenial/man5/drbd.conf.5.html](https://manpages.ubuntu.com/manpages/xenial/man5/drbd.conf.5.html)
- [manpages.ubuntu.com/manpages/jammy/man8/drbdadm.8.html](https://manpages.ubuntu.com/manpages/jammy/man8/drbdadm.8.html)
- [ubuntu.com/server/docs/distributed-replicated-block-device-drbd](https://ubuntu.com/server/docs/distributed-replicated-block-device-drbd)