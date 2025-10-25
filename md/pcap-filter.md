---
title: Pcap Filter
---


### Ether
```go
  
Offset | 00  01  02  03  04  05  06  07  08  09  0A  0B  0C  0D
-------+--------------------------------------------------------
0000   | 00  16  3E  C5  6C  94  00  0C  29  C3  
        \___________/  \___________/  \________/
          Dest MAC         Src MAC       Proto
  
```

| | |
|-------------|-------------|
| `ether src MAC` | Filter by source address |
| `ether dst MAC` | Filter by destination address |
| `ether host MAC` | Ether source or destination matches |
| `ether proto arp` | Filter proto ARP |
| `ether proto 0x88ca` | Filter proto TIPC (0x88ca) |

<br>
