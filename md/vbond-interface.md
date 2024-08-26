---
title:  'Bond Interface'
---

# 1. Introduction
A bond interface is a virtual network interface that combines multiple network interfaces into a single one. It ensures that if one network interface fails, others can take over, and systems remain connected.
  
Create a bond interface.
```sh

$ ip link add <interface-name> type bond
  
```
- *interface-name*: the name assigned to the interface.

Add interfaces to the bond interface.  
```sh
  
$ ip link set <another_interface> master <bond_interface>
  
```
  
Set bond mode.  
```sh
  
$ ip link set <interface-name> type bond mode <bond_mode>
  
```
  
- Bond modes:  
    - **balance-rr**: Packets are transmitted by the Round-Robin algorithm across all slave interfaces.
    - **active-backup**: Only one slave is active at a time. Another slave becomes active if the currently active slave fails.
    - **balance-xor**: Transmits based on a hash algorithm, usually involving the source and destination MAC addresses.
    - **broadcast**: Transmits packets to all slave interfaces.
    - **balance-tlb**: Adaptive Transmit Load Balancing. Dynamically adjusts the transmission load among slave interfaces based on the current traffic.
    - **balance-alb**: Adaptive Load Balancing. Includes the features of `balance-tlb` plus receiving load balancing.
    - **802.3ad**: Link Aggregation Control Protocol (LACP).

# 2. Labs
## 2.1. Create a Bond Interface

<script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
    mermaid.initialize({
        look: 'handDrawn',
        theme: 'neutral',
    });
</script>

<pre class="mermaid">
flowchart LR
    subgraph "bond interface"
        bond0
        dummy1
        dummy2
    end

    network <---> bond0
    bond0 <-- primay --> dummy1
    bond0 <-. backup .-> dummy2
</pre>


- Create dummy interfaces: dummy1, dummy2
- Create a bond interface: bond0
- Add dummy1, dummy2 to bond0.

```sh

# create dummy interfaces
$ ip link add dummy1 type dummy
$ ip link add dummy2 type dummy

# create bond interface
$ ip link add bond0 type bond

# set bond mode
$ ip link set bond0 type bond mode active-backup

# add dummy1, dummy2 to bond0
$ ip link set dummy1 master bond0
$ ip link set dummy2 master bond0

# assign IP address to bond0
$ ip addr add 192.168.1.3/24 dev bond0

# turn on interfaces
$ ip link set dummy1 up
$ ip link set dummy2 up
$ ip link set bond0 up
  
```

- Check the bond0 interface.

```sh
   
$ ping -I bond0 192.168.1.3
$ cat /proc/net/bonding/bond0
   
```


- Redundancy test: turn off a interface.
```sh
  
# turn of primary interface and ping
$ ip link set dummy1 down
$ cat /proc/net/bonding/bond0
$ ping -I bond0 192.168.1.3
  
```

# References
- https://man7.org/linux/man-pages/man8/ip-link.8.html
- https://www.kernel.org/doc/Documentation/networking/bonding.txt