---
weight: 1
title: "VETH"
bookToc: false
bookFlatSection: true
---

# Virtual Ethernet Interface
## 1. Introduction
- A virtual ethernet interface (veth) acts as tunnels between devices.
- Packets transmitted on one device in the pair are immediately received on the other device.

## 2. Demo
### 2.1. Connect Namespaces
- Create two namespaces: ns1, ns2.
- Connect them by a veth pair: veth1-veth2.
- Check the connection with ping.

{{< mermaid >}}
flowchart LR
    subgraph ns1
        veth1
    end
    subgraph ns2
        veth2
    end

    veth1 --- veth2
{{< /mermaid >}}


```sh
# create namepsaces
$ ip netns add ns1
$ ip netns add ns2

# create veth pair
$ ip link add veth1 type veth peer name veth2

# assign each end of the pair to a namespace
$ ip link set veth1 netns ns1
$ ip link set veth2 netns ns2

# assign IP for each end of the pair
$ ip netns exec ns1 ip addr add 192.168.1.10/24 dev eth1
$ ip netns exec ns1 ip link set veth1 up

$ ip netns exec ns1 ip addr add 192.168.1.20/24 dev eth2
$ ip netns exec ns1 ip link set veth2 up

# check the connection with ping
$ ip netns exec ns1 ping 192.168.1.20
$ ip netns exec ns2 ping 192.168.1.10
```


## X. References
- https://man7.org/linux/man-pages/man8/ip-link.8.html
- https://man7.org/linux/man-pages/man4/veth.4.html
- https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking