---
title:  'Virtual IP Address'
---

# 1. Introduction
A Virtual IP address (VIP) is an IP address shared among multiple devices. If one device fails, another takes over, ensuring the address remains active.


# 2. Setup Virtual IP with Keepalived.
**Agenda**

- Build a Docker image with Keepalived installed.
- Create two containers: node1, node2.
- Configure virtual IP on node1 as master.
- Configure virtual IP on node2 as backup.
- Failover test:
    - Shutdown node1.
    - Verify that the virtual IP is automatically reassigned to node2.


## 2.1. Build Docker Image.
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


## 2.2. Setup Virtual IP on Node1.
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

Start syslog to monitor keepalived activity.
```sh
  
$ service rsyslog start
  
```

Start keepalived.
```sh
  
$ service keepalived start
  
```

Check /var/log/syslog
```sh
  
node1 Keepalived[80]: Starting Keepalived v2.0.19 (10/19,2019)
node1 Keepalived[80]: Running on Linux 6.8.0-47-generic #47-Ubuntu SMP PREEMPT_DYNAMIC Fri Sep 27 21:40:26 UTC 2024 (built for Linux 5.4.166)
node1 Keepalived[80]: Command line: '/usr/sbin/keepalived'
node1 Keepalived[80]: Opening file '/etc/keepalived/keepalived.conf'.
node1 Keepalived[81]: Starting VRRP child process, pid=82
node1 Keepalived_vrrp[82]: Registering Kernel netlink reflector
node1 Keepalived_vrrp[82]: Registering Kernel netlink command channel
node1 Keepalived_vrrp[82]: Opening file '/etc/keepalived/keepalived.conf'.
node1 Keepalived_vrrp[82]: (VI_1) Warning - nopreempt will not work with initial state MASTER - clearing
node1 Keepalived_vrrp[82]: Registering gratuitous ARP shared channel
node1 Keepalived_vrrp[82]: (VI_1) Entering BACKUP STATE (init)
node1 Keepalived_vrrp[82]: (VI_1) Entering MASTER STATE
  
```

Check IP address.
```sh
  
$ ip addr show eth0
12: eth0@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 192.168.200.11/24 scope global eth0
       valid_lft forever preferred_lft forever
  
```
- The address 192.168.200.11/24 is added to interface eth0.


## 2.3. Setup Virtual IP on Node2.
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

Start syslog to monitor keepalived activity.
```sh
  
$ service rsyslog start
  
```

Start keepalived.
```sh
  
$ service keepalived start
  
```

Check /var/log/syslog
```sh
  
node2 Keepalived[57]: Starting Keepalived v2.0.19 (10/19,2019)
node2 Keepalived[57]: Running on Linux 6.8.0-47-generic #47-Ubuntu SMP PREEMPT_DYNAMIC Fri Sep 27 21:40:26 UTC 2024 (built for Linux 5.4.166)
node2 Keepalived[57]: Command line: '/usr/sbin/keepalived'
node2 Keepalived[57]: Opening file '/etc/keepalived/keepalived.conf'.
node2 Keepalived[58]: Starting VRRP child process, pid=59
node2 Keepalived_vrrp[59]: Registering Kernel netlink reflector
node2 Keepalived_vrrp[59]: Registering Kernel netlink command channel
node2 Keepalived_vrrp[59]: Opening file '/etc/keepalived/keepalived.conf'.
node2 Keepalived_vrrp[59]: Registering gratuitous ARP shared channel
node2 Keepalived_vrrp[59]: (VI_1) Entering BACKUP STATE (init)
  
```

Check IP address.
```sh
  
$ ip addr show eth0
14: eth0@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
  
```
- Currently, node2 in backup state. So, the address 192.168.200.11/24 still not exist.


# References
- https://man7.org/linux/man-pages/man8/ip-link.8.html
- https://man7.org/linux/man-pages/man8/ip-netns.8.html
- https://man7.org/linux/man-pages/man4/veth.4.html