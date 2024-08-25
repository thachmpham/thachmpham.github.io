---
title:  'Dummy Interface'
---

# 1. Introduction
A dummy interface is a virtual network interface that operates entirely in software. It supports to configure IP addresses and network setttings without requiring physical network hardware.
  
A dummy interface could be similar to the following things:  
- A loopback interface.

Create a dummy interface.
```sh

$ ip link add <interface-name> type dummy
  
```
- interface-name: the name assigned to the interface.

# 2. Labs
## 2.1. Create a Dummy Interface
- Create a dummy interface: dummy0
- Assign IP and turn on the interface.

```sh
# create interface
$ ip link add dummy0 type dummy

# assign IP and turn on
$ ip addr add 192.168.1.100/24 dev dummy0
$ ip link set dummy0 up

# show interface info
$ ip addr show dummy0
  
```


# References
- https://man7.org/linux/man-pages/man8/ip-link.8.html
