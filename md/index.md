---
title:  "thachmpham.github.io"
subtitle: "A Software Engineer's Notebook<br/>*Computer Networking - Distributed Systems - Software Debugging*"
---


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

<br>

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

| Programing |
|-----------------|
| [Convert Program to High Availability](html/opensaf-amf-non-sa-aware.html) |
| [Build High Availability Program](html/opensaf-amf-sa-aware.html) |
| [Healthcheck](html/opensaf-healthcheck.html) |
| [Error Detection & Recovery](html/opensaf-amf-error-detection.html) |

:::
::::::::::::::

## Software Debugging
### GDB
:::::::::::::: {.columns}
::: {.column}

| Usage |
|-----------------|
| [Find Memory](html/gdb-find.html) |
| [Dump Restore Memory](html/gdb-dump-restore.html) |
| [Call Functions](html/gdb-call.html) |
| [Pretty Printer](html/gdb-write-pp.html) |
| [STL Containers](html/gdb-stl.html) |

:::
::: {.column}

| Debug |
|-----------------|
| [Script Progrm](html/gdb-program-started-by-script.html) |
| [Systemd Program](html/gdb-program-started-by-systemd.html) |
| [OpenSAF Program](html/gdb-ha-program.html) |
| [Examine ELF File](html/elf.html) |
| [Virtual Address & EFL Offset](html/virtual-addr-elf-offset.html) |
| [C Stack under Assembly](html/asm-callstack.html) |
| [Override Libc Functions](html/ld_preload.html) |
| [Multiple Threads](html/gdb-multithread.html) |
| [Reproduce Eagain Socket Error](html/tipc_eagain.html) |

:::
::::::::::::::

<br>

| Linux Kernel | |
|-----------------|----------------:|
| [Build & Run](html/kernel_build.html) | *(buildroot qemu)* |
| [Debug Kernel with GDB](html/kernel_debug_gdb.html) | *(kernel qemu gdb)* |
| [User Space to Kernel Space](html/kernel_user_to_kernel.html) | *(kernel qemu gdb)* |

<br>


## Cheatsheets
:::::::::::::: {.columns}
::: {.column}

| Debug |
|-----------------|
|[gdb](html/gdb.html) |
|[bashdb](html/bashdb.html) |

:::
::: {.column}

| Network |
|-----------------|
|[tcpdump](html/tcpdump.html) |
|[pcap-filter](html/pcap-filter.html) |

:::
::: {.column}

| Text |
|-----------------|
|[grep](html/grep.html)|
|[perl](html/perl.html)|
|[awk](html/awk.html)|

:::
::: {.column}

| Diagram |
|-----------------|
|[plantuml](html/plantuml.html)|
|[graphviz](html/graphviz.html)|

:::
::: {.column}

| Virtualization |
|-----------------|
|[kvm](html/kvm.html)|
|[docker](html/docker.html)|

:::
::::::::::::::


:::::::::::::: {.columns}
::: {.column}

| Linux |
|-----------------|
|[disk](html/fdisk.html)|
|[systemd](html/systemd.html)|
|[firewall](html/firewall.html) |

:::
::::::::::::::

