---
title:  'Virtual IP Address'
---

# 1. Introduction
A Virtual IP address (VIP) is an IP address shared among multiple devices. If one device fails, another takes over, ensuring the address remains active.

Some tools to setup a virtual IP:

- Keepalived.  
- Linux Heartbeat.  


# 2. Lab
Use **keepalived** to setup a virtual IP on two nodes:

- Build keepalived docker image.
- Create two containers node 1 and node 2.
- Setup a virtual IP shared between node1 and node2.
    - Node 1 is master. The virtual IP will assigned to node 1.
    - Node 2 is backup. If node 1 stops, the virtual IP will automatically switch to node 2.


## 2.1. Build Docker Image
- Dockerfile.
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

- Build image.
```sh
  
$ docker build --progress=plain -t keepalived .
  
```


## 2.2. Setup Node 1
- Create container.
```sh
  
$ docker run --privileged -d --name node1 --hostname node1 keepalived
  
```

- Access terminal.
```sh
  
$ docker exec -it node1 bash
  
```

- Edit /etc/keepalived/keepalived.conf
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

- Start keepalived.
```sh
  
node1$ service keepalived start
  
```

- Show IP.
```sh
  
node1$ ip addr show eth0
eth0    UP      172.17.0.2/16   192.168.200.11/24

# node 1 was master, so the virtual IP 192.168.200.11 assigned to it.
  
```


## 2.3. Setup Node 2
- Create container.
```sh
  
$ docker run --privileged -d --name node2 --hostname node2 keepalived
  
```

- Access terminal.
```sh
  
$ docker exec -it node2 bash
  
```

- Edit /etc/keepalived/keepalived.conf
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

- Start keepalived.
```sh
  
node1$ service keepalived start
  
```

- Show IP.
```sh
  
node1$ ip addr show eth0
eth0    UP      172.17.0.3/16

# the virtual IP not shown, because node 2 was backup.
  
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

    docker0 <---> if1
    docker0 <---> if2

</pre>


## 2.3. Fault Tolerance
- Stop keepalived on node 1.
```sh
  
node1$ service keepalived stop

node1$ ip -brief addr show eth0
eth0    UP      172.17.0.2/16

# the virtual IP is removed from node1.
  
```

- Check IP on node 2.
```sh
  
node2$ ip -brief addr show eth0
eth0    UP      172.17.0.3/16   192.168.200.11/24

# after node 1 stopped, the virtual IP  automatically assigned to node 2.
  
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

- Check pcap.
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
- Setup route to allow ping from host to container.
```sh
  
host$ sudo ip route add 192.168.200.0/24 dev docker0
  
```

- Ping the virtual IP when node 1 is master.
```sh
  
host$ sudo arping -C 1 -i docker0 192.168.200.11
42 bytes from 02:42:ac:11:00:02 (192.168.200.11): index=0 time=9.639 usec

# when node 1 was master, ping got MAC of node 1.
  
```

- Ping the virtual IP when node 2 is master.
```sh
  
host$ sudo arping -C 1 -i docker0 192.168.200.11
42 bytes from 02:42:ac:11:00:03 (192.168.200.11): index=0 time=7.195 usec

# when node 2 was master, ping got MAC of node 2.
  
``` 


# 3. Troubleshooting
- Check log of keepalived.
```sh
  
$ service rsyslog start
$ tail -f /var/log/syslog
  
```


# References
- [keepalived.org](https://www.keepalived.org)
- [keepalived.readthedocs.io](https://keepalived.readthedocs.io/en/latest)