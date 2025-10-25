---
title: Pcap Filter
---


### Ether
| Offset | Header Field                | Size          |
|:-------:|----------------------------|:--------------|
| 0x0000  | Destination MAC            | 6 bytes       |
| 0x0006  | Source MAC                 | 6 bytes       |
| 0x000C  | EtherType                  | 2 bytes       |
|         | **Header size**            | **14 bytes** |


| Filter | |
|-------------|-------------|
| `ether src MAC` | Filter by source address |
| `ether dst MAC` | Filter by destination address |
| `ether host MAC` | Ether source or destination matches |
| `ether proto arp` | Filter proto ARP |
| `ether proto 0x88ca` | Filter proto TIPC (0x88ca) |

<br>
