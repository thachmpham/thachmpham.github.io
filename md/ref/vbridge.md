---
title:  'Virtual Bridge Interface'
---

# 1. Introduction
A virtual bridge connects multiple network interfaces together, allowing them to communicate with each other as if they were on the same network.  
  
A virtual bridge can simulate following things:  
- A switch.  
- A bridge.  


Create a virtual bridge.
```sh
  
$ ip link add <bridge_name> type bridge
  
```

Set the bridge as the master devive of another one.
```sh
  
$ ip link set <another_device> master <bridge_device>
  
```

Example:

```sh
# Create a virtual bridge: virbr0
$ ip link add virbr0 type bridge

# Create veth devices: veth0, veth1, veth2
$ ip link add veth0 type veth
$ ip link add veth1 type veth peer name veth2

# Connect veth0, veth1 to virbr0
$ ip link set veth0 master virbr0
$ ip link set veth1 master virbr0
  
```

<script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
    mermaid.initialize({
        look: 'handDrawn',
        theme: 'neutral',
    });
</script>

<pre class="mermaid">
flowchart TD
    veth0 <---> virbr0
    veth2 <---> veth1
    veth1 <---> virbr0
</pre>


# 2. Labs
## 2.1. Connect Namespaces
- Create namespaces: ns1, ns2.
- Create veth pairs: veth1-vethbr1, veth2-vethbr2.
- Move veth1 to ns1, veth2 to ns2.
- Create a bridge: virbr0.
- Connect vethbr1, vethbr2 to virbr0.

<pre class="mermaid">
flowchart TD
    subgraph ns1
        veth1
    end
    subgraph ns2
        veth2
    end    
    veth1 <---> vethbr1
    veth2 <---> vethbr2    
    vethbr1 <---> virbr0
    vethbr2 <---> virbr0    
</pre>


```sh
  
# create namepsaces
$ ip netns add ns1
$ ip netns add ns2

# create veth pairs
$ ip link add veth1 type veth peer name vethbr1
$ ip link add veth2 type veth peer name vethbr2

# move veth endpoints to namespaces
$ ip link set veth1 netns ns1
$ ip link set veth2 netns ns2

# create virtual bridge
$ ip link add virbr0 type bridge

# connect veth endpoints to bridge
$ ip link set vethbr1 master virbr0
$ ip link set vethbr2 master virbr0

# assign IP
$ ip netns exec ns1 ip addr add 192.168.0.10/24 dev veth1
$ ip netns exec ns2 ip addr add 192.168.0.20/24 dev veth2

# turn on devices
$ ip netns exec ns1 ip link set veth1 up
$ ip netns exec ns2 ip link set veth2 up
$ ip link set vethbr1 up
$ ip link set vethbr2 up
$ ip link set virbr0 up

# show learned mac addressed of the bridge
$ brctl showmacs virbr0

# check connection with ping
$ ip netns exec ns1 ping 192.168.0.20
$ ip netns exec ns2 ping 192.168.0.10
  
```


## 2.2. Connect Docker Containers
- Create containers: container1, container2.
- Create veth pairs: veth1-vethbr1, veth2-vethbr2.
- Move veth1 to container1, veth2 to container2.
- Create a bridge: virbr0.
- Connect vethbr1, vethbr2 to virbr0.

<pre class="mermaid">
flowchart TD
    subgraph container1
        veth1
    end
    subgraph container2
        veth2
    end    
    veth1 <---> vethbr1
    veth2 <---> vethbr2    
    vethbr1 <---> virbr0
    vethbr2 <---> virbr0    
</pre>


```sh
  
# create containers
$ docker run -id --cap-add=NET_ADMIN --name container1 alpine sh
$ docker run -id --cap-add=NET_ADMIN --name container2 alpine sh

# create veth pairs
$ ip link add veth1 type veth peer name vethbr1
$ ip link add veth2 type veth peer name vethbr2

# move veth1 to namespace of container1
$ pid_of_container1=$(docker inspect --format '{{.State.Pid}}' container1)
$ ip link set veth1 netns ${pid_of_container1}

# move veth2 to namespace of container2
$ pid_of_container2=$(docker inspect --format '{{.State.Pid}}' container2)
$ ip link set veth2 netns ${pid_of_container2}

# create virtual bridge
$ ip link add virbr0 type bridge

# connect veth endpoints to bridge
$ ip link set vethbr1 master virbr0
$ ip link set vethbr2 master virbr0

# assign IP
$ docker exec container1 ip addr add 192.168.0.10/24 dev veth1
$ docker exec container2 ip addr add 192.168.0.20/24 dev veth2

# turn on devices
$ docker exec container1 ip link set veth1 up
$ docker exec container2 ip link set veth2 up
$ ip link set vethbr1 up
$ ip link set vethbr2 up
$ ip link set virbr0 up

# check connection with ping
$ docker exec container1 ping 192.168.0.20
$ docker exec container2 ping 192.168.0.10
  
```

## 3.3. Split Network with VLAN.
#### Agenda
- Setup connected network.  
    - Create namespaces: ns1, ns2, ns3.
    - Create veth pairs: veth1-vethbr1, veth2-vethbr2, veth3-vethbr3.
    - Move veth1 to ns1, veth2 to ns2, veth3 to ns3.
    - Create a bridge: br0.
    - Connect vethbr1, vethbr2, vethbr3 to br0.
- Split the network.
    - Move vethbr1, vethbr2 to vlan 10.
    - Move vethbr3 to vlan 20.
    - vethbr1, vethbr2 are on the same vlan. So, they can ping each other.
    - vethbr3 is on the different vlan. So, vethbr3 cannot ping to vethbr1, vethbr2 and vise versa.

### 3.3.1. Setup Connected Network.
```sh
  
# create namepsaces
$ ip netns add ns1
$ ip netns add ns2
$ ip netns add ns3

# create veth pairs
$ ip link add veth1 type veth peer name vethbr1
$ ip link add veth2 type veth peer name vethbr2
$ ip link add veth3 type veth peer name vethbr3

# move veth endpoints to namespaces
$ ip link set veth1 netns ns1
$ ip link set veth2 netns ns2
$ ip link set veth3 netns ns3

# create virtual bridge
$ ip link add br0 type bridge

# connect veth endpoints to bridge
$ ip link set vethbr1 master br0
$ ip link set vethbr2 master br0
$ ip link set vethbr3 master br0

# assign IP
$ ip netns exec ns1 ip addr add 192.168.0.10/24 dev veth1
$ ip netns exec ns2 ip addr add 192.168.0.20/24 dev veth2
$ ip netns exec ns3 ip addr add 192.168.0.30/24 dev veth3

# turn on devices
$ ip netns exec ns1 ip link set veth1 up
$ ip netns exec ns2 ip link set veth2 up
$ ip netns exec ns3 ip link set veth3 up
$ ip link set vethbr1 up
$ ip link set vethbr2 up
$ ip link set vethbr3 up
$ ip link set br0 up

# show interfaces of br0
$ brctl show br0
bridge name     bridge id               STP enabled     interfaces
br0             8000.9e4287f72992       no              vethbr1
                                                        vethbr2
                                                        vethbr3
  
```

Show vlan configuration.
```sh
  
$ bridge vlan show
port              vlan-id
vethbr1           1 PVID Egress Untagged
vethbr2           1 PVID Egress Untagged
vethbr3           1 PVID Egress Untagged
br0               1 PVID Egress Untagged
  
```

<pre class="mermaid">
flowchart TD
    subgraph ns1
        veth1
    end
    subgraph ns2
        veth2
    end    
    subgraph ns3
        veth3
    end  
    veth1 <---> vethbr1
    veth2 <---> vethbr2    
    veth3 <---> vethbr3    
    vethbr1 <--->|vlan 1| br0
    vethbr2 <--->|vlan 1| br0
    vethbr3 <--->|vlan 1| br0
</pre>

- By default, vethbr1, vethbr2, vethbr3 belong to the same vlan (vid=1). So, they can ping each other.

```sh
  
# ns1 -> ns2
$ ip netns exec ns1 ping -c 1 192.168.0.20
64 bytes from 192.168.0.20: icmp_seq=1 ttl=64 time=0.058 ms

# ns1 -> ns3
$ ip netns exec ns1 ping -c 1 192.168.0.30
64 bytes from 192.168.0.30: icmp_seq=1 ttl=64 time=0.119 ms
  
```


### 3.3.2. Split Network.
<pre class="mermaid">
flowchart TD
    subgraph ns1
        veth1
    end
    subgraph ns2
        veth2
    end    
    subgraph ns3
        veth3
    end  
    veth1 <---> vethbr1
    veth2 <---> vethbr2    
    veth3 <---> vethbr3    
    vethbr1 <--->|vlan 10| br0
    vethbr2 <--->|vlan 10| br0
    vethbr3 <--->|vlan 20| br0
</pre>

```sh
  
# enable vlan filtering
$ ip link set dev br0 type bridge vlan_filtering 1

# check vlan filtering
$ ip -details link show br0
br0:
    vlan_filtering 1
  
```


```sh
  
# move vethbr1, vethbr2 to vlan 10
$ bridge vlan add vid 10 dev vethbr1 pvid untagged
$ bridge vlan add vid 10 dev vethbr2 pvid untagged

# move vethbr1 to vlan 20
$ bridge vlan add vid 20 dev vethbr3 pvid untagged

# show vlan config
$ bridge vlan show
port              vlan-id  
vethbr1           1 Egress Untagged
                  10 PVID Egress Untagged
vethbr2           1 Egress Untagged
                  10 PVID Egress Untagged
vethbr3           1 Egress Untagged
                  20 PVID Egress Untagged
br0               1 PVID Egress Untagged
  
```

```sh
  
# remove vlan 1
$ bridge vlan del vid 1 dev vethbr1
$ bridge vlan del vid 1 dev vethbr2
$ bridge vlan del vid 1 dev vethbr3

# show vlan config
$ bridge vlan show
vethbr1           10 PVID Egress Untagged
vethbr2           10 PVID Egress Untagged
vethbr3           20 PVID Egress Untagged
  
```

```sh
  
# ns1 -> ns2: ping ok
$ ip netns exec ns1 ping -c 1 192.168.0.20
64 bytes from 192.168.0.20: icmp_seq=1 ttl=64 time=0.062 ms

# ns1 -> ns3: cannot ping
$ ip netns exec ns1 ping -c 1 192.168.0.30
1 packets transmitted, 0 received, 100% packet loss, time 0ms
  
```


# References
- https://man7.org/linux/man-pages/man8/ip-link.8.html
- https://man7.org/linux/man-pages/man8/ip-netns.8.html