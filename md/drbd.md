---
title: Distributed Replicated Block Device
---

## 1. Introduction
Distributed Replicated Block Device (DRBD) is a tool for data replication at the block level between servers.


## 2. Setup DRBD
Set up DRBD to replicate partition /dev/sda9 between two hosts node1 and node2.

<pre class="mermaid">
flowchart LR
    node1_drbd0["/dev/drbd0"]
    node2_drbd0["/dev/drbd0"]

    subgraph node1
        subgraph node1_sda9["/dev/sda9"]
            node1_drbd0
        end
    end

    subgraph node2
        subgraph node2_sda9["/dev/sda9"]
            node2_drbd0
        end
    end

    node1_drbd0 <---> node2_drbd0
</pre>

- /dev/sda9 are the existing partitions in nodes.
- /dev/drbd0 are the abstract partitions which will be created by DRBD.
- /dev/drbd0 between two nodes are sync with each other.


### 2.1. Install DRBD
- Install DRBD in two nodes.
```sh
  
$ apt install drbd-utils  
  
```


### 2.2. Configuration File
- Edit files /etc/drbd.conf on node1 and node2 as below.
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


### 2.3. Create meta-data
- Create meta-data on node1.
```sh
  
node1$ drbdadm create-md r0
  
```

- Create meta-data on node2.
```sh
  
node2$ drbdadm create-md r0
  
```

### 2.4. Start service
- Start drbd service on node1.
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

- Start drbd service on node2.
```sh
  
node2$ systemctl start drbd
  
```

- After drbd started, node2 connects to node1.
```sh
  
# syslog
#   start service
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

#   connect to node1
node2 kernel:   drbd r0:    Handshake sucessfull
node2 kernel:   drbd r0:    conn( WFConnection  ->  WFReportParams )
node2 kernel:   block r0:   peer( Unknown -> Secondary ), conn( WFReportParams -> Connected ), pdsk( DUnknown -> Inconsistent )
node2 systemds: Finish drbd.service
  
```

```sh
  
# syslog
#   connect to node1
node2 kernel:   drbd r0:    Handshake sucessfull
node2 kernel:   drbd r0:    conn( WFConnection  ->  WFReportParams )
node2 kernel:   block r0:   peer( Unknown -> Secondary ), conn( WFReportParams -> Connected ), pdsk( DUnknown -> Inconsistent )
node2 systemds: Finish drbd.service
  
```


### 2.5. Set primary
- Promote node1 to be primary, overwrite the current state of node2.
```sh
  
$ drbdadm -- --overwrite-date-of-peer primary all
  
```

- node1 becomes sync source.
```sh
  
# syslog
node1 kernel:   block drbd0:    role( Secondary  ->  Primary )   disk( Inconsistent    ->  UpToDate )
node1 kernel:   block drbd0:    conn( Connected  ->  WFBitMapS )
node1 kernel:   block drbd0:    conn( WFBitMapS  ->  SyncSource )
  
```

- node2 becomes sync target.
```sh
  
# syslog
node2 kernel:   block drbd0:    peer( Secondary  ->  Primary )   disk( Inconsistent    ->  UpToDate )
node2 kernel:   block drbd0:    conn( Connected  ->  WFBitMapT )
node2 kernel:   block drbd0:    conn( WFBitMapT  ->  WFSyncUUID )
node2 kernel:   block drbd0:    conn( WFSyncUUID ->  SyncTarget )
  
```

- sync done.
```sh
  
# syslog
node1 kernel:   block drbd0:    conn( SyncSource  ->  Connected )   pdsk (Inconsistent  ->  UpToDate )
  
```

```sh
  
# syslog
node2 kernel:   block drbd0:    conn( SyncTarget  ->  Connected )   pdsk (Inconsistent  ->  UpToDate )
  
```

- Command to monitor during sync progress.
```sh
  
$ watch -n1 cat /proc/drbd
  
```

- Check status
```sh
  
node1$ drbdadm status
r0  role:Primary
    disk:UpToDate
    peer    role:Secondary
            replication:Established     peer-disk:UpToDate
  
```

```sh
  
node2$ drbdadm status
r0  role:Secondary
    disk:UpToDate
    peer    role:Primary
            replication:Established     peer-disk:UpToDate
  
```


### 2.7. Add filesystem
- After sync completed, the partition /dev/drbd0 will be created.
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

- Make filesystem.
```sh
  
$ mkfs.ext3 /dev/drbd0
  
```


### 2.7. Mount
- On node1, mount partition /dev/drbd0 to /mnt/test (*only primary node can mount*).
```sh
  
node1$ mkdir /mnt/test

node1$ mount /dev/drbd0 /mnt/test
  
```


## 3. Fault Tolerance
- On node1, create a file /mnt/test/hello.
```sh
  
node1$ touch /mnt/test/hello
  
```

- Stop drbd on node1.
```sh
  
node1$ systemctl stop drbd
  
```

```sh
  
node1 systemd:  Stopping drbd.service
node1 kernel:   EXT4-fs (drbd0): unmounting filesystem
node1 drbd:     Stopping all DRBD resources
node1 kernel:   block drbd0:    role( Primary   ->  Secondary )
node1 kernel:   drbd r0:        peer( Secondary ->  Unknown ), conn(Connected   ->  Disconnecting ) pdsk( UpToDate   ->  DUnknown )
node1 kernel:   drbd r0:        conn( Disconnecting ->  StandAlone )
node1 kernel:   drbd r0:        disk( UpToDate  ->  Failed )
node1 kernel:   drbd r0:        disk( Failed    ->  Diskless )
node1 systemds: Stopped drbd.service
  
```

```sh
  
node2 kernel:   block drbd0:    peer( Primary   ->  Secondary)
node2 kernel:   drbd r0:        peer( Secondary ->  Unknown ) conn( Connected   ->  TearDown ) pdsk( UpToDate   ->  DUnknown )
node2 kernel:   drbd r0:        conn( TearDown  ->  Unconnected )
node2 kernel:   drbd r0:        conn( Unconnected  ->  WFConnection )
  
```

- Promote node2 becomes primary. (*Need to promote to Primary before mount*).
```sh
  
node2$ drbdadm primary r0
  
```

```sh
  
node2 kernel: block drbd0: role( Secondary  ->  Primary )  
  
```

- Mount.
```sh
  
node$ mount /dev/drbd0 /mnt/test
  
```

- The /mnt/test/hello file will exist on node2.
```sh
  
node2$ ls /mnt/test
hello
  
```


# References
- https://manpages.ubuntu.com/manpages/xenial/man5/drbd.conf.5.html
- https://manpages.ubuntu.com/manpages/jammy/man8/drbdadm.8.html
- https://ubuntu.com/server/docs/distributed-replicated-block-device-drbd