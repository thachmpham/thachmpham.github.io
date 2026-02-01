---
title:  'IP Routing'
---

# 1. Introduction
IP routing is the method by which networks route data packets from a source to destination.

Set default route.
```sh
  
$ ip route add <destination_network> via <gateway_IP> dev <interface>
  
```

Add, remove a route.
```sh
  
$ ip route add <destination_network> dev <interface>
$ ip route del <destination_network>
  
```

Show routing table.
```sh

$ ip route  
  
```

# 2. Labs
## 2.1. Destination Directly Connected.
If the destination IP address matches a network directly connected to the router, the packet is delivered directly over that network link.

<pre class="mermaid">
flowchart LR
    subgraph ns1[namespace ns1]
        veth1[192.168.0.1]
    end

    subgraph ns2[namespace ns2]
        veth2[192.168.0.2]
    end   

    subgraph net-br0[bridge br0]
        br0[192.168.0.0/24]
    end

    br0 <---> veth1
    br0 <---> veth2

</pre>

```sh
  
# create namespaces
$ ip netns add ns1
$ ip netns add ns2

# create virtual bridge
$ ip link add br0 type bridge
$ ip link set br0 up

# connect namespace 1 to the bridge
$ ip link add veth1 type veth peer name veth1-br
$ ip link set veth1 netns ns1
$ ip link set veth1-br master br0
$ ip link set veth1-br up
  
# connect namespace 2 to the bridge
$ ip link add veth2 type veth peer name veth2-br
$ ip link set veth2 netns ns2
$ ip link set veth2-br master br0
$ ip link set veth2-br up


# set IP addresses
$ ip netns exec ns1 ip addr add 192.168.0.1/24 dev veth1
$ ip netns exec ns1 ip link set veth1 up

$ ip netns exec ns2 ip addr add 192.168.0.2/24 dev veth2
$ ip netns exec ns2 ip link set veth2 up
  
```

From ns1, ping to ns2.
```sh
  
$ ip netns exec ns1 ping -c 1 -R 192.168.0.2
  
```


## 2.2. Destination in Routing Table.
If there is a route for the destination IP address in the routing table, the packet is sent to the next-hop address listed in the table.


<pre class="mermaid">
flowchart LR
    subgraph ns3[namespace ns3]
        veth3[192.168.0.3]
    end

    subgraph ns4[namespace ns4]
        veth4[10.0.0.4]
    end   

    subgraph net-br0[bridge br1]
        veth3-br[192.168.0.0/24]
        veth4-br[10.10.0.0/24]
    end

    veth3-br <---> veth3
    veth4-br <---> veth4

</pre>

```sh
  
# create namespaces
$ ip netns add ns3
$ ip netns add ns4

# create virtual bridge
$ ip link add br1 type bridge
$ ip link set br1 up

# connect ns3 and br1
$ ip link add veth3 type veth peer name veth3-br
$ ip link set veth3 netns ns3
$ ip link set veth3-br master br1
$ ip link set veth3-br up
  
# connect ns4 and br1
$ ip link add veth4 type veth peer name veth4-br
$ ip link set veth4 netns ns4
$ ip link set veth4-br master br1
$ ip link set veth4-br up


# set IP addresses
$ ip netns exec ns3 ip addr add 192.168.0.3/24 dev veth3
$ ip netns exec ns3 ip link set veth3 up

$ ip netns exec ns4 ip addr add 10.0.0.4/24 dev veth4
$ ip netns exec ns4 ip link set veth4 up
  
```

Setup routing table.
```sh
  
# Instruct ns3 to go to ns4 through veth3
# ns3 --> veth3 --> veth3-br --> veth4-br --> veth4 --> ns4
$ ip netns exec ns3 ip route add 10.0.0.0/24 dev veth3

# Instruct ns4 to go to ns3 through veth4
# ns4 --> veth4 --> veth4-br --> veth3-br --> veth3 --> ns3
$ ip netns exec ns4 ip route add 192.168.0.0/24 dev veth4
  
```

From ns3, ping to ns4.
```sh
  
$ ip netns exec ns3 ping -c 1 -R 10.0.0.4
  
```

- Output
```sh

PING 192.168.0.3 (192.168.0.3) 56(124) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=0.063 ms
RR:     10.0.0.4
        192.168.0.3
        192.168.0.3
        10.0.0.4


--- 192.168.0.3 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.063/0.063/0.063/0.000 ms
  
```

Reset routing table for next steps.
```sh
  
$ ip netns exec ns3 ip route delete 10.0.0.0/24
$ ip netns exec ns4 ip route delete 192.168.0.0/24
  
```

## 2.3. Forward through Default Route.
If no specific route is found, but a default route exists, the packet is forwarded to the default route.

Setup default route.
```sh
  
# instruct ns3 to use veth3 as default route
# ns3 --> veth3 --> ...
$ ip netns exec ns3 ip route add default via 192.168.0.3 dev veth3

# instruct ns4 to use veth4 as default route
# ns4 --> veth4 --> ...
$ ip netns exec ns4 ip route add default via 10.0.0.4 dev veth4
  
```

From ns3, ping to ns4.
```sh
  
$ ip netns exec ns3 ping -c 1 -R 10.0.0.4
  
```


# References
- https://man7.org/linux/man-pages/man8/ip-route.8.html