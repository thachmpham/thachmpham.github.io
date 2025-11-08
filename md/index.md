---
title:  "thachmpham.github.io"
subtitle: "A Software Engineer's Notebook<br/>*Computer Networking - Distributed Systems - Software Debugging*"
---

## Software Debugging
### GDB
:::::::::::::: {.columns}
::: {.column}

| Examine Data |
|-----------------|
| [Variables](html/gdb/variable.html)|
| [Memory](html/gdb/memory.html)|
| [Pretty Printers](html/gdb/pretty-printer.html)|

<br>

| Examine Execution |
|-----------------|
| [Breakpoints](html/gdb/breakpoint.html)|
| [Catchpoints](html/gdb/catchpoint.html)|
| [Watchpoints](html/gdb/watchpoint.html)|

<br>

| Examine Binary |
|-----------------|
| [Symbols](html/gdb/symbol.html)|
| <br> |
| <br> |

:::
::: {.column}

| Remote Debug |
|-----------------|
| [GDBServer](html/gdb/gdbserver.html) |
| [Tracepoints](html/gdb/tracepoint.html) |
| <br> |

<br>

| Best Practice |
|-----------------|
| [Systemd Program](html/gdb-program-started-by-systemd.html) |
| [OpenSAF Program](html/gdb-ha-program.html) |
| [Multiple Threads](html/gdb-multithread.html) |

:::
::::::::::::::


### Linux
:::::::::::::: {.columns}
::: {.column}

| Kernel |
|-----------------|
| [Build & Run](html/kernel_build.html) |
| [Debug Kernel with GDB](html/kernel_debug_gdb.html) |
| [User Space to Kernel Space](html/kernel_user_to_kernel.html) |

:::
::: {.column}

| Binary Analysis |
|-----------------|
| [Examine ELF File](html/elf.html) |
| [Virtual Address & EFL Offset](html/virtual-addr-elf-offset.html) |
| [C to Assembly](html/asm-callstack.html) |
| [LD_PRELOAD](html/ld_preload.html) |

:::
::::::::::::::

| Reproduce |
|-----------------|
| [Socket EAGAIN](html/tipc_eagain.html) |


## Networking
:::::::::::::: {.columns}
::: {.column}

| Protocols | |
|-----------------|----------------:|
| [ARP](html/arp.html) | *(arping)* |
| [IP Route](html/ip-routing.html) | *(iproute2)* |
| [DHCP](html/dhcp.html) | *(kea-dhcp4)* |
| [Virtual IP](html/vip.html) | *(keepalived)* |
| [PXE](html/pxe.html) | *(dnsmasq)* |
| [LDAP](html/ldap.html) | *(openldap)* |
| [DRBD](html/drbd.html) | *(drbd-utils)* |
| [TIPC](html/tipc.html) | *(tipcutils)* |

:::
::: {.column}

| Virtual Interfaces | |
|-----------------|----------------:|
| [Virtual Ethernet](html/veth.html) | (*ip veth*) |
| [Virtual Bridge](html/vbridge.html) | *(ip bridge)* |
| [Dummy Interface](html/vdummy-interface.html) | *(ip dummy)* |
| [Bond Interface](html/vbond-interface.html) | *(ip bond)* |

:::
::::::::::::::


## Distributed Systems
### OpenSAF
:::::::::::::: {.columns}
::: {.column}

| Setup |
|-----------------|
| [Build & Install](html/opensaf-install.html) |
| [Setup Cluster 1-SC](html/opensaf-1sc.html) |
| [Setup Cluster 2-SC](html/opensaf-2sc.html) |

:::
::: {.column}

| Service |
|-----------------|
| [IMM](html/opensaf-imm.html) |
| [SMF](html/opensaf-smf.html) |
| [LOG](html/opensaf-log.html) |

:::
::: {.column}

| Development |
|-----------------|
| [Convert Program to High Availability](html/opensaf-amf-non-sa-aware.html) |
| [Build High Availability Program](html/opensaf-amf-sa-aware.html) |
| [Healthcheck](html/opensaf-healthcheck.html) |
| [Error Detection & Recovery](html/opensaf-amf-error-detection.html) |

:::
::::::::::::::


## Cheatsheets
:::::::::::::: {.columns}
::: {.column}

| |
|-----------------|
|[gdb](html/gdb.html) |
|[bashdb](html/bashdb.html) |
|[tcpdump](html/tcpdump.html) |
|[pcap-filter](html/pcap-filter.html) |

:::
::: {.column}

| |
|-----------------|
|[grep](html/grep.html)|
|[perl](html/perl.html)|
|[awk](html/awk.html)|

:::
::: {.column}

| |
|-----------------|
|[plantuml](html/plantuml.html)|
|[graphviz](html/graphviz.html)|

:::
::: {.column}

| |
|-----------------|
|[kvm](html/kvm.html)|
|[docker](html/docker.html)|
|[disk](html/fdisk.html)|
|[systemd](html/systemd.html)|
|[firewall](html/firewall.html) |

:::
::::::::::::::


