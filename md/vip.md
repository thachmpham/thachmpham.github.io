---
title:  'Virtual IP Address'
---

# 1. Introduction
A Virtual IP address (VIP) is an IP address shared among multiple devices. If one device fails, another takes over, ensuring the address remains active.


# 2. Virtual IP with Keepalived
**Agenda**

- Build a Docker image for Keepalived.
- Create two containers named node1 and node2.
- Configure a virtual IP on node1 to act as the master.
- Configure the same virtual IP on node2 to act as the backup.
- After configuration, the virtual IP will be active on node1. If node1 stops, the virtual IP will automatically switch to node2.


## 2.1. Build Docker Image
Create Dockerfile.
```Dockerfile
  
FROM ubuntu:20.04

# skip questions during install keepalived
ENV DEBIAN_FRONTEND=noninteractive

RUN apt update
RUN apt install -y keepalived 
RUN apt install -y rsyslog vim tshark

# keep container running
CMD ["tail", "-f", "/dev/null"]
  
```

Build docker image.
```sh
  
$ docker build --progress=plain -t keepalived .
  
```


## 2.2. Setup Node1
Create container node1.
```sh
  
$ docker run --privileged -d --name node1 --hostname node1 keepalived
  
```

Access to node1.
```sh
  
$ docker exec -it node1 bash
  
```

Edit /etc/keepalived/keepalived.conf
```python
  
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 50
    nopreempt
    priority 100
    advert_int 1
    virtual_ipaddress {
        192.168.200.11/24
    }
}
  
```

Start keepalived.
```sh
  
node1$ service keepalived start
  
```

The virtual IP 192.168.200.11 will be assigned to node1.
```sh
  
node1$ ip addr show eth0
eth0    UP      172.17.0.2/16   192.168.200.11/24
  
```


## 2.3. Setup Node2
Create container node2.
```sh
  
$ docker run --privileged -d --name node2 --hostname node2 keepalived
  
```

Access to node1.
```sh
  
$ docker exec -it node2 bash
  
```

Edit /etc/keepalived/keepalived.conf
```python
  
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 50
    nopreempt
    priority 100
    advert_int 1
    virtual_ipaddress {
        192.168.200.11/24
    }
}
  
```

Start keepalived.
```sh
  
node1$ service keepalived start
  
```

Currently, node2 in backup state. So, the virtual IP still not exist.
```sh
  
node1$ ip addr show eth0
eth0    UP      172.17.0.3/16
  
```

<pre class="mermaid">
flowchart LR
    if1[eth0]
    if2[eth0]

    subgraph node1
        if1 <--> 172.17.0.2
        if1 <--> 192.168.200.11
    end 

    subgraph node2
        if2 <--> 172.17.0.3
    end

    docker0 <---> ARPING 192.168.200.11if1
    docker0 <---> if2

</pre>


## 2.3. Fault Tolerance
Stop keepalived on node1.
```sh
  
node1$ service keepalived stop
  
```

The virtual IP is removed from node1.
```sh
  
node1$ ip -brief addr show eth0
eth0    UP      172.17.0.2/16
  
```


After stop keepalived on node1, the instance on node2 becomes master, so the virtual IP is ressigned to node2.
```sh
  
node2$ ip -brief addr show eth0
eth0    UP      172.17.0.3/16   192.168.200.11/24
  
```

<pre class="mermaid">
flowchart LR
    if1[eth0]
    if2[eth0]

    subgraph node1
        if1 <--> 172.17.0.2
    end

    subgraph node2
        if2 <--> 172.17.0.3
        if2 <--> 192.168.200.11
    end 

    docker0 <---> if1
    docker0 <---> if2

</pre>

Analyze pcap packets.
```sh
  
node1$ tshark -i eth0

# master node sends VRRP Announcement to backup nodes to inform that master is still alive
# 224.0.0.18 is a multicast address for nodes in the VRRP group.
# node1 -> multicast: VRRP Announcement
172.17.0.2 ? 224.0.0.18   VRRP 54 Announcement (v2)

# master node sends IGMPv3 Membership Report / Leave group to backup nodes to inform that master leaves
# node1 -> multicast: IGMP Membership Report / Leave group
172.17.0.2 ? 224.0.0.22   IGMPv3 54 Membership Report / Leave group 224.0.0.18

# node2 informs that the virtual IP is assigned to it
# node2 become master
# node2 -> multicast: ARP Gratuitous
02:42:ac:11:00:03 ? Broadcast    ARP 42 Gratuitous ARP for 192.168.200.11 (Request)

# # node2 -> multicast: VRRP Announcement
172.17.0.3 ? 224.0.0.18   VRRP 54 Announcement (v2)
  
```


## 2.4. Ping
Setup route.
```sh
  
host$ sudo ip route add 192.168.200.0/24 dev docker0
  
```

Ping to the virtual IP when node1 is master, we receive MAC address of node1.
```sh
  
host$ sudo arping -C 1 -i docker0 192.168.200.11
42 bytes from 02:42:ac:11:00:02 (192.168.200.11): index=0 time=9.639 usec
  
```

Ping to the virtual IP when node2 is master, we receive MAC address of node2.
```sh
  
host$ sudo arping -C 1 -i docker0 192.168.200.11
42 bytes from 02:42:ac:11:00:03 (192.168.200.11): index=0 time=7.195 usec
  
``` 


# 3. Troubleshooting
Enable syslog and monitor the log of Keepalived.
```sh
  
$ service rsyslog start
$ tail -f /var/log/syslog
  
```


# References
- https://www.keepalived.org