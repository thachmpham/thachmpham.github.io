---
title:  'Two-Phase Commit'
---


# 1. Introduction
The Dynamic Host Configuration Protocol (DHCP) is a network management protocol used to dynamically assign IP addresses to devices on a network.  

<script>
  d3.select("body").append("p").text("wow, impressed")
</script>


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