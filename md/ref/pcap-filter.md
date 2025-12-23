---
title: Pcap Filter
---


# Ether

:::::::::::::: {.columns}
::: {.column width=50%}

| Offset | Header Field     | Size    | Filter    |
|:-----|:-------------------|:-------:|:----------|
| 0x00  | Destination MAC   | 6       | `ether dst` |
| 0x06  | Source MAC        | 6       | `ether src` |
| 0x0C  | EtherType         | 2       | `ether proto` |
|       |                   |         |           |
|       | **Header Size**   | **14**  |           |

:::
::: {.column width=50%}

```go
                              Ethernet II Header

      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x00  |                      Destination MAC Address                  |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x04  |          Destination MAC (cont.)           |  Source MAC      |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x08  |                      Source MAC (cont.)                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x0C  |           EtherType           |                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x0E  |                              Payload                          |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x..  |                               FCS                             |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

:::
::::::::::::::

# IP

:::::::::::::: {.columns}
::: {.column width=50%}

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

:::
::: {.column width=50%}

```go
                                 IPv4 Header

      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x00  | Ver |  IHL  |   DSCP   | ECN  |        Total Length           |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x04  |        Identification         |Flags|   Fragment Offset       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x08  |  Time to Live |    Protocol   |        Header Checksum        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x0C  |                       Source Address                          |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x10  |                    Destination Address                        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x14  |                    Options (if IHL > 5)                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x..  |                              Data                             |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

:::
::::::::::::::

# TCP

:::::::::::::: {.columns}
::: {.column width=50%}

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

:::
::: {.column width=50%}

```go
                                  TCP Header

      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x00  |          Source Port          |       Destination Port        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x04  |                        Sequence Number                        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x08  |                    Acknowledgment Number                      |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x0C  | Data |       |C|E|U|A|P|R|S|F|                                |
      | Off  | Resvd |W|C|R|C|S|S|Y|I|        Window Size             |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x10  |           Checksum            |         Urgent Pointer        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x14  |                    Options (if Data Offset > 5)               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x..  |                              Data                             |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

:::
::::::::::::::

# UDP

:::::::::::::: {.columns}
::: {.column width=50%}

| Offset | Field        | Size | Pcap Filter |
|--------|--------------|:----:|-------------|
| 0x00   | Source Port  | 2    | `udp src port` |
| 0x02   | Destination Port | 2| `udp dst port` |
| 0x04   | Length       | 2    | `udp[4:2]`   |
| 0x06   | Checksum     | 2    | `udp[6:2]`   |
|        |              |          |     |
|        | **Header Size** | **8** |     |

:::
::: {.column width=50%}

```go
                                  UDP Header

      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x00  |          Source Port          |       Destination Port        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x04  |            Length             |           Checksum            |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x08  |                              Data                             |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

:::
::::::::::::::

# TIPC

:::::::::::::: {.columns}
::: {.column width=50%}

| Field        | Size   | Ether  | UDP    |
|--------------|:------:|--------|--------|
| Version      |  3b    | `ether[0x0e] >> 5` |        |
| User         |  4b    | `ether[0x0e] >> 1 & 0x0f` |        |
| Hsize        |  4b    | `ether[0x0e:2] >> 5 & 0x0f` |        |
| NDSR         |  4b    |        |        |
| Msize        |  13b   |        |        |
| Mtype        |  3b    | `ether[0x10] >> 5` |        |


| User | Name                | Purpose                          | Class      |
|:----:|:--------------------|:---------------------------------|:----------:|
| 0    | LOW_IMPORTANCE      | Low Importance Data              | Payload    |
| 1    | MEDIUM_IMPORTANCE   | Medium Importance Data           | Payload    |
| 2    | HIGH_IMPORTANCE     | High Importance Data             | Payload    |
| 3    | CRITICAL_IMPORTANCE | Critical Importance Data         | Payload    |
| 4    | USER_TYPE_4         | Reserved for future use          | N/A        |
| 5    | BCAST_PROTOCOL      | Broadcast Link Protocol          | Internal   |
| 6    | MSG_BUNDLER         | Message Bundler Protocol         | Internal   |
| 7    | LINK_PROTOCOL       | Link State Protocol              | Internal   |
| 8    | CONN_MANAGER        | Connection Manager               | Internal   |
| 9    | USER_TYPE_9         | Reserved for future use          | N/A        |
| 10   | CHANGEOVER_PROTOCOL | Link Changeover Protocol         | Internal   |
| 11   | NAME_DISTRIBUTOR    | Name Table Update Protocol       | Internal   |
| 12   | MSG_FRAGMENTER      | Msg Fragmentation Protocol       | Internal   |
| 13   | LINK_DISCOVER       | Neighbor Detection Protocol      | Internal   |
| 14   | USER_TYPE_14        | Reserved for future use          | N/A        |
| 15   | USER_TYPE_15        | Reserved for future use          | N/A        |


| User/Class   | Mtype  |  Name  |
|-------:|:-----:|:--------------|
| Payload | 0     | CONN_MSG    |
| Payload | 1     | MCAST_MSG   |
| Payload | 2     | NAMED_MSG   |
| Payload | 3     | DIRECT_MSG  |
| NAME_DISTRIBUTOR   | 0     | Name publication   |
| NAME_DISTRIBUTOR   | 1     | Name withdrawl     |
| LINK_DISCOVER      | 0     | Link request          |
| LINK_DISCOVER      | 1     | Link response         |


```go
                     TIPC Neighbor Discovery Message
    
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x00  | Ver | User  | Hsize |N|R|R|R|           Message Size          |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x04  |Mtype|        Capabilities       |      Node Signature         |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x08  |                      Destination Domain                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x0C  |                         Previous Node                         |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x10  |                          Network Id                           |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x14  |                                                               |
      +-+-+-+-+-+-+-                                    +-+-+-+-+-+-+-+
0x18  |                         Media Address                         |
      +-+-+-+-+-+-+-                                    +-+-+-+-+-+-+-+
0x1C  |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x20  |                           Reserved                            |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x24  |                           Reserved                            |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x28  |                           Reserved                            |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```


:::
::: {.column width=50%}

```go
                        TIPC Payload Message
    
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x00  | Ver | User  | Hsize |N|D|S|R|          Message Size           |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x04  |Mtype| Error |Reroute|Lsc| RES |     Broadcast Acknowledge     |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x08  |        Link Acknowledge       |        Link Sequence          |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x0C  |                         Previous Node                         |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x10  |                        Originating Port                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x14  |              Destination Port / Destination Network           |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x18  |                        Originating Node                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x1C  |                        Destination Node                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x20  |                            Name Type                          |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x24  |                 Name Instance / Name Sequence Lower           |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x28  |                      Name Sequence Upper                      |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x2C  |                             Data                              |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                  Version - Destination Port/Network: Required.
      Originating Node - Name Sequence Upper: Conditional. Data: Optional.
```


```go
                  TIPC Name Table Distributor Message  
    
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x00  | Ver | User  | Hsize |N|R|R|R|          Message Size           |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x04  |Mtype|     RESERVED            |     Broadcast Acknowledge     |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x08  |        Link Acknowledge       |        Link Sequence          |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x0C  |                         Previous Node                         |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x10  |                        Originating Port                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x14  |              Destination Port / Destination Network           |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x18  |                        Originating Node                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x1C  |                       Destination Node                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x20  |                            RESERVED                           |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x24  |   Item Size   |M|           RESERVED                          |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x28  |                             Data                              |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

```go
                     TIPC Link Internal Message
    
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x00  |vers |msg usr|hdr sz |n|resrv|            packet size          |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x04  |m typ|   sequence gap          |       broadcast ack no        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x08  |link level ack no/bc gap after | link level/bc seqno/bc gap to |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x0C  |                       previous node                           |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x10  |  last sent broadcast/fragm no | next sent pkt/ fragm msg no   |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x14  |          session no           | res |r|berid|link prio|netpl|p|
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x18  |                      originating node                         |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x1C  |                      destination node                         |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x20  |                  transport sequence number                    |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x24  |   msg count/max packet        |       link tolerance          |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0x28  |                             Data                              |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

:::
::::::::::::::