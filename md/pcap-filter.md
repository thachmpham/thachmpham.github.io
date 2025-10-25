---
title: Pcap Filter
---


### Ether
| Offset | Header Field     | Size    | Filter    |
|:-----|:-------------------|:-------:|:----------|
| 0x00  | Destination MAC   | 6       | `ether dst` |
| 0x06  | Source MAC        | 6       | `ether src` |
| 0x0C  | EtherType         | 2       | `ether proto` |
|       | **Header Size**   | **14**  |           |

<br>

### IP
| Offset | Header Field     | Size| Filter |
|:-----|:-------------------|:----|:-----------------------|
| 0x00 | Version (*4b*)     |     | `ip[0x00] >> 4 & 0x0f` |
|      | IHL (*4b*)         | 1   | `ip[0x00] & 0x0f` |
| 0x01 | Type of Service    | 1   | `ip[0x01]` |
| 0x02 | Total Length       | 2   | `ip[0x02:2]` |
| 0x04 | Identification     | 2   | `ip[0x04:2]` |
| 0x06 | Flags (*3b*)       |     | `ip[0x06] >> 5 & 0x0f` |
|      | FragOffset (*13b*) | 2   | `ip[0x06:2] & 0x1fff` |
| 0x08 | Time To Live       | 1   | `ip[0x08]` |
| 0x09 | Protocol           | 1   | `ip proto` |
| 0x0A | Header Checksum    | 2   | `ip[0x0a:2]` |
| 0x0C | Source IP          | 4   | `ip src` |
| 0x10 | Destination IP     | 4   | `ip dst` |
| 0x14 | Options (if IHL > 5) | * |         |
|      | **Header Size**    | **4\*IHL**| |

