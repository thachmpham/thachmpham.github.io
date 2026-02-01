---
title:  'Dynamic Host Configuration Protocol'
---


# 1. Introduction
The Dynamic Host Configuration Protocol (DHCP) is a network management protocol used to dynamically assign IP addresses to devices on a network.  

DHCP operates based on a client-server model.

- DHCP server is responsibles for assigning, renewing, and releasing IP addresses to devices that request them.
- DHCP client is any device that requests an IP address from the DHCP server.

Start an IPv4 DHCP server.
```sh
  
$ kea-dhcp4 -c <config_file>
  
```

Send a DHCP request.
```sh
  
$ dhclient <network_interface>
  
```


# 2. Labs
- Setup a DHCP server on host.
- Setup namespaces `ns1`, `ns2` as DHCP clients.
- From `ns1`, `ns2`, send DHCP requests to the server.

<pre class="mermaid">
flowchart LR
    subgraph host
        dhcp-server
    end

    subgraph br0[bridge: br0]
        veth1-br
        veth2-br
    end

    subgraph ns1[namespace: ns1]
        veth1
    end

    subgraph ns2[namespace: ns2]
        veth2
    end

    host <---> br0
    veth1-br <--->|veth pair| veth1
    veth2-br <--->|veth pair| veth2
</pre>


## 2.1. Setup DHCP Server
Install isc-kea.
```sh
  
$ apt install kea
  
```

Setup virtual bridge interface `br0`.
```sh
  
# create bridge
$ ip link add br0 type bridge

# set IP
$ ip addr add 192.168.1.1/24 dev br0

# turn on
$ ip link set br0 up
  
```

Edit file `/etc/kea/kea-dhcp4.conf`.
```json
  
{
  "Dhcp4": {
	"interfaces-config": {
  		"interfaces": [ "br0" ]
	},
	"lease-database": {
    	"type": "memfile"
	},
	"subnet4": [
  	{
    	"id": 1,
    	"subnet": "192.168.1.0/24",
    	"pools": [
			{
				"pool": "192.168.1.150 - 192.168.1.200"
			}
    	],
  	}
	]
  }
}
  
```

Start DHCP server.
```sh
  
$ systemctl restart kea-dhcp4-server
  
```

Check status
```sh
  
$ systemctl status kea-dhcp4-server
  
```


## 2.2. Setup Client.
Setup namespace `ns1`.
```sh
  
# create namespace
$ ip netns add ns1


# below steps to connect ns1 and host

# create veth pair
$ ip link add veth1 type veth peer name veth1-br

# move veth1 to ns1
$ ip link set veth1 netns ns1

# move veth1-br to br0
$ ip link set veth1-br master br0

# turn on veth pair
$ ip link set veth1-br up
$ ip netns exec ns1 ip link set veth1 up
  
```


## 2.2. Send DHCP Request.
From `ns1`, send a DHCP request.
```sh
  
$ ip netns exec ns1 dhclient veth1
  
```

Check IP addresses of `ns1`.
```sh
   
$ ip netns exec ns1 ip addr show
  
```

- Output
```sh
  
  veth1@if8:
    inet 192.168.1.150/24 brd 192.168.1.255 scope global dynamic veth1
  
```

*For namespace `ns2`, we do the same steps as `ns1` to setup and send DHCP requests.*


## 2.3. Remove Assigned IP Address.
From `ns1`, remove the assigned IP address.
```sh
  
$ ip netns exec ns1 dhclient -r veth1
  
```

## 2.4. Capture DHCP messages.
DHCP operates over UDP ports 67 (server) and 68 (client).
```sh
  
$ tshark -P -i br0 -f "udp port 67 or udp port 68" -w dhcp.pcap
  
```

```sh
  
$ ip netns exec ns1 dhclient veth1
  
```


## 2.5. Analyze DHCP messages.
Read pcap.
```sh
  
$ tshark -r dhcp.pcap
  
```

- Output
```sh
  
1 0.000000000      0.0.0.0 → 255.255.255.255  DHCP 342 DHCP Discover  - Transaction ID 0x62535106
2 0.001860896  192.168.1.1 → 192.168.1.150    DHCP 342 DHCP Offer     - Transaction ID 0x62535106
3 0.003278980      0.0.0.0 → 255.255.255.255  DHCP 342 DHCP Request   - Transaction ID 0x62535106
4 0.003918114  192.168.1.1 → 192.168.1.150    DHCP 342 DHCP ACK       - Transaction ID 0x62535106
  
```

# Tips
Observe live log of `kea-dhcp4-server` service.
```sh
  
$ journalctl -f -u kea-dhcp4-server
  
```

# References
- https://kea.readthedocs.io/en/kea-2.0.1/arm/dhcp4-srv.html
- https://kea.readthedocs.io/en/kea-1.6.2/man/kea-dhcp4.8.html
- https://linux.die.net/man/8/dhclient
- https://ubuntu.com/server/docs/how-to-install-and-configure-isc-kea