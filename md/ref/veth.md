---
title:  'Virtual Ethernet Interface'
---

# 1. Introduction
A virtual ethernet interface (**veth**) acts as tunnels between devices. Packets transmitted on one device in the pair are immediately received on the other device.  
  
A veth could be similar to the following things in real life:  
- A cable connecting two computers.  
- A cable connecting between switches, bridges.  
  
Create a veth pair.
```sh
  
$ ip link add <p1-name> type veth peer name <p2-name>
  
```
- *p1-name* and *p2-name* are the names assigned to the two connected end points.

# 2. Labs
## 2.1. Connect Two Namespaces
- Create two namespaces: ns1, ns2.
- Connect them by a veth pair: veth1-veth2.
- Check the connection with ping.

<script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
    mermaid.initialize({
        look: 'handDrawn',
        theme: 'neutral',
    });
</script>

<pre class="mermaid">
flowchart LR
    subgraph ns1
        veth1
    end
    subgraph ns2
        veth2
    end

    veth1 <---> veth2
</pre>


```sh
# create namepsaces: ns1, ns2
$ ip netns add ns1
$ ip netns add ns2

# create veth pair: veth1-veth2
$ ip link add veth1 type veth peer name veth2

# move veth1 to namespace ns1
$ ip link set veth1 netns ns1

# move veth2 to namespace ns2
$ ip link set veth2 netns ns2

# set IP for veth1
$ ip netns exec ns1 ip addr add 192.168.0.10/24 dev veth1
$ ip netns exec ns1 ip link set veth1 up

# set IP for veth2
$ ip netns exec ns2 ip addr add 192.168.0.20/24 dev veth2
$ ip netns exec ns2 ip link set veth2 up

# check connection with ping
$ ip netns exec ns1 ping 192.168.0.20
$ ip netns exec ns2 ping 192.168.0.10
  
```

## 2.2. Connect Two Containers
- Create docker containers: container1, container2.
- Connect them by a veth pair: veth1-veth2.
- Check the connection with ping.

<pre class="mermaid">
flowchart LR
    subgraph container1
        veth1
    end
    subgraph container2
        veth2
    end

    veth1 <---> veth2
</pre>


```sh
# create containers
$ docker run -id --cap-add=NET_ADMIN --name container1 alpine sh
$ docker run -id --cap-add=NET_ADMIN --name container2 alpine sh

# create veth pair: veth1-veth2
$ ip link add veth1 type veth peer name veth2

# move veth1 to namespace of container1
$ pid_of_container1=$(docker inspect --format '{{.State.Pid}}' container1)
$ ip link set veth1 netns ${pid_of_container1}

# move veth2 to namespace of container2
$ pid_of_container2=$(docker inspect --format '{{.State.Pid}}' container2)
$ ip link set veth2 netns ${pid_of_container2}

# set IP for veth1
$ docker exec container1 ip addr add 192.168.0.10/24 dev veth1
$ docker exec container1 ip link set veth1 up

# set IP for veth2
$ docker exec container2 ip addr add 192.168.0.20/24 dev veth2
$ docker exec container2 ip link set veth2 up

# check connection with ping
$ docker exec container1 ping 192.168.0.20
$ docker exec container2 ping 192.168.0.10
  
```

# References
- https://man7.org/linux/man-pages/man8/ip-link.8.html
- https://man7.org/linux/man-pages/man8/ip-netns.8.html
- https://man7.org/linux/man-pages/man4/veth.4.html