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
</script>

<pre class="mermaid">
flowchart TD
    veth0 --- virbr0
    veth2 --- veth1
    veth1 --- virbr0
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
    veth1 --- vethbr1
    veth2 --- vethbr2    
    vethbr1 --- virbr0
    vethbr2 --- virbr0    
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
    veth1 --- vethbr1
    veth2 --- vethbr2    
    vethbr1 --- virbr0
    vethbr2 --- virbr0    
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

# References
- https://man7.org/linux/man-pages/man8/ip-link.8.html
- https://man7.org/linux/man-pages/man8/ip-netns.8.html