---
weight: 1
title: "ARP"
bookToc: true
bookFlatSection: true
---

# ARP
## 1. Introduction
The Address Resolution Protocol (ARP) resolves **IP addresses** to **MAC addresses** in a local network, ensuring that data is sent to the correct physical device.

## 2. How ARP Works
Given a network consisting of 4 nodes and a router as below.

{{< mermaid >}}
flowchart TD        
    Node1["Node1
    IP: 192.168.0.10
    MAC: 00:00:00:00:00:10"]

    Node2["Node2
    IP: 192.168.0.20
    MAC: 00:00:00:00:00:20"]

    Node3["Node3
    IP: 192.168.0.30
    MAC: 00:00:00:00:00:30"]

    Node4["Node4
    IP: 192.168.0.40
    MAC: 00:00:00:00:00:40"]


    Node1 --- Router
    Node2 --- Router
    Node3 --- Router
    Node4 --- Router
{{< /mermaid >}}


Let's explore how Node1 discovers the MAC address of Node4 who has IP address 192.168.0.40.

{{< mermaid >}}
sequenceDiagram   
    Node1 ->> Router: Who has 192.168.0.40
    Router ->> Node2: Who has 192.168.0.40
    Router ->> Node3: Who has 192.168.0.40
    Router ->> Node4: Who has 192.168.0.40
    Node4 ->> Router: 192.168.0.40 is at 00:00:00:00:00:40
    Router ->> Node1: 192.168.0.40 is at 00:00:00:00:00:40
{{< /mermaid >}}

-   Node1 sends the message "Who has 192.168.0.40?" to Router.
-   Router sends the message "Who has 192.168.0.40?" to Node2.  
    Node2 compares its IP and 192.168.0.40. The IPs are different, so Node2 ignore the message.
-   Router sends the message "Who has 192.168.0.40?" to Node3.  
    Node3 compares its IP and 192.168.0.40. The IPs are different, so Node3 ignore the message.
-   Router sends the message "Who has 192.168.0.40?" to Node4.  
    Node4 compares its IP and 192.168.0.40. The IPs are the same, so Node4 replies the message with its MAC address.

## 3. Send & Capture ARP Packets.
Node1: Check network interfaces.
```sh
$ ifconfig
wlo1:   flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.106  netmask 255.255.255.0  broadcast 192.168.1.255
```

Node1: Capture ARP packets.
```sh
$ sudo arping -v -i wlo1 -C 1 192.168.1.27
```

Node1: Send an ARP request.
```sh
$ sudo tshark -i wlo1 -f "arp" -w capture.pcap
```

Node1: Read the packets.
```sh
$ sudo tshark -r capture.pcap
No  Time        Source                  Destination             Protocol    Len     Info
1	0.000000000	ChongqingFug_67:5c:c5	Broadcast	            ARP	        58	    Who has 192.168.1.27? Tell 192.168.1.106
2	0.093148373	66:84:ef:bc:c6:25	    ChongqingFug_67:5c:c5	ARP	        60	    192.168.1.27 is at 66:84:ef:bc:c6:25
```

## 4. ARP Table
