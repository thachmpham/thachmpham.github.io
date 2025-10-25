---
title: Pcap Filter
---


### Ether
| Offset | Header Field     | Size    | Filter    |
|:-----|:-------------------|:-------:|:----------|
| 0x00  | Destination MAC   | 6       | `ether dst` |
| 0x06  | Source MAC        | 6       | `ether src` |
| 0x0C  | EtherType         | 2       | `ether proto` |
|       |                   |         |           |
|       | **Header Size**   | **14**  |           |

<br>

### IP
| Offset | Header Field     | Size| Filter |
|:-----|:-------------------|:----:|:-----------------------|
| 0x00 | Version (*4b*)     |     | `ip[0x00] >> 4` |
|      | IHL (*4b*)         | 1   | `ip[0x00] & 0x0f` |
| 0x01 | Type of Service    | 1   | `ip[0x01]` |
| 0x02 | Total Length       | 2   | `ip[0x02:2]` |
| 0x04 | Identification     | 2   | `ip[0x04:2]` |
| 0x06 | Flags (*3b*)       |     | `ip[0x06] >> 5 & 0x0f` |
|      | Frag Offset (*13b*)| 2   | `ip[0x06:2] & 0x1fff` |
| 0x08 | Time To Live       | 1   | `ip[0x08]` |
| 0x09 | Protocol           | 1   | `ip proto` |
| 0x0A | Header Checksum    | 2   | `ip[0x0a:2]` |
| 0x0C | Source IP          | 4   | `ip src` |
| 0x10 | Destination IP     | 4   | `ip dst` |
| 0x14 | Options            | *   | *if IHL > 5* |
|      |                    |             |         |
|      | **Header Size**    | **20 - 60** | **4 * IHL** |

<br>

### TCP

| Offset | Field            | Size | Filter |
|--------|----------------------|:----:|--------------------|
| 0x00   | Source Port          | 2    | `tcp src port` |
| 0x02   | Destination Port     | 2    | `tcp dst port` |
| 0x04   | Sequence Number      | 4    | `tcp[0x04:4]` |
| 0x08   | Acknowledgment Number | 4   | `tcp[0x08:4]` |
| 0x0C   | Data Offset (*4b*)    |     | `tcp[0x0c] >> 4` |
|        | Reserved (*3b*)      |      | `tcp[0x0c] >> 1 & 0x07` |
|        | NS flag (*1b*)       | 1    | `tcp[0x0c] & 0x01` |
| 0x0D   | Flags                | 1    | `tcp[0x0d]` |
| 0x0E   | Window Size          | 2    | `tcp[0x0e:2]` |
| 0x10   | Checksum             | 2    | `tcp[0x10:2]` |
| 0x12   | Urgent Pointer       | 2    | `tcp[0x12:2]` |
| 0x14   | Options              | *    | *if Data Offset > 5* |
|        |                      |             |        |
|        | **Header Size**      | **20 - 60** |  **4 * DataOffset** |

<br>

### UDP

| Offset | Field        | Size | Pcap Filter |
|--------|--------------|------|-------------|
| 0x00   | Source Port  | 2    | `udp src port` |
| 0x02   | Destination Port | 2| `udp dst port` |
| 0x04   | Length       | 2    | `udp[4:2]`   |
| 0x06   | Checksum     | 2    | `udp[6:2]`   |
|        |              |          |     |
|        | **Header Size** | **8** |     |

<br>

### TIPC
```go
  
    3 3 2 2 2 2 2 2 2 2 2 2 1 1 1 1 1 1 1 1 1 1 0 0 0 0 0 0 0 0 0 0
    1 0 9 8 7 6 5 4|3 2 1 0 9 8 7 6|5 4 3 2 1 0 9 8|7 6 5 4 3 2 1 0
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
w0:| Ver | User  | Hsize |N|D|S|R|          Message Size           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
w1:|Mtype| Error |Reroute|Lsc| RES |     Broadcast Acknowledge     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
w2:|        Link Acknowledge       |        Link Sequence          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
w3:|                         Previous Node                         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
w4:|                        Originating Port                       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
w5:|              Destination Port / Destination Network           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
w6:|                        Originating Node                       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
w7:|                        Destination Node                       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
w8:|                            Name Type                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
w9:|                 Name Instance / Name Sequence Lower           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
wA:|                      Name Sequence Upper                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   \                                                               \
   /                                                               /
   \                             Data                              \
   /                                                               /
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

        w0-w5: Required. w6-wA: Conditional. Data: Optional
  
```

| Field | Size | Ether  | UDP  |
|-------|------|--------|------|
|       |   |   |   |