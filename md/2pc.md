---
title:  'Two-Phase Commit'
---


# 1. Introduction
The Dynamic Host Configuration Protocol (DHCP) is a network management protocol used to dynamically assign IP addresses to devices on a network.  


::: {#prepare}
:::

<script>
// Select the div and append an SVG container
const svg = d3.select("#prepare")
  .append("svg")
  .attr("width", 400)
  .attr("height", 400);

// Add a circle to the SVG container
svg.append("circle")
  .attr("cx", 200)    // x-position of the center
  .attr("cy", 200)    // y-position of the center
  .attr("r", 50)      // radius
  .attr("fill", "blue");  // color of the circle
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

::: {#commit}
:::

<script>
// Select the div and append an SVG container
const svg2 = d3.select("#commit")
  .append("svg")
  .attr("width", 400)
  .attr("height", 400);

// Add a circle to the SVG container
svg2.append("circle")
  .attr("cx", 200)    // x-position of the center
  .attr("cy", 200)    // y-position of the center
  .attr("r", 50)      // radius
  .attr("fill", "red");  // color of the circle
</script>
