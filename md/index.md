---
title:  "thachmpham.github.io"
subtitle: "A Software Engineer's Notebook<br/>*Computer Networking - Distributed Systems - Software Debugging*"
---


## 1. Networking
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


## 2. Distributed Systems
### 2.1. OpenSAF
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
| [Build High Availability Application](html/opensaf-amf-sa-aware.html) |
| [Healthcheck](html/opensaf-healthcheck.html) |
| [Error Detection & Recovery](html/opensaf-amf-error-detection.html) |

:::
::::::::::::::

<br>

## 3. Software Debugging
### 3.1. C/C++
| |
|:-----------------|----------------:|
| [Debug a Program Started by a Script](html/gdb-program-started-by-script.html)    | *(gdb catch)*     |
| [Debug A Program Started by SystemD](html/gdb-program-started-by-systemd.html)    | *(gdb remote)*    |
| [Debug a High Availability Program](html/gdb-ha-program.html) | *(gdb non-stop)* |
| [Find Memory Inside Running Process](html/gdb-find.html)                   | *(gdb find)*  |
| [Dump & Restore Process Memory](html/gdb-dump-restore.html)   | *(gdb dump restore)*  |
| [Execute Functions on the Fly](html/gdb-call.html)               | *(gdb call)*  |
| [Examine Contents of ELF File](html/elf.html)                    | *(gdb objdump)*  |
| [Virtual Address & EFL Offset](html/virtual-addr-elf-offset.html)        | *(gdb sharedlibrary)*   |
| [Inspect C Stack with Assembly](html/asm-callstack.html) | *(gdb asm reg)* |
| [Print Contents of STL Containers](html/gdb-stl.html)             | *(gdb pretty-printer)*   |
| [Method for Printing Complex Data](html/gdb-write-pp.html)             | *(gdb pretty-printer)*   |
| [Modify Return Values of Libc Functions](html/ld_preload.html) | *(ld_preload dlsym)* |
| [GDB & Multiple Threads](html/gdb-multithread.html) | *(gdb thread non-stop async)* |


### 3.2. Linux
| |
|:-----------------|----------------:|
| [Build & Run Linux Kernel](html/kernel_build.html) | *(buildroot qemu)* |
| [Debug Linux Kernel with GDB](html/kernel_debug_gdb.html) | *(kernel qemu gdb)* |
| [From User Space to Kernel Space](html/kernel_user_to_kernel.html) | *(kernel qemu gdb)* |
| [Reproduce Eagain Socket Error](html/tipc_eagain.html) | *(tipc socket buffer)* |

<br>


## 4. Cheatsheets
:::::::::::::: {.columns}

::: {.column}
| Debug |
|-----------------|
|[gdb](html/gdb.html) |
|[bashdb](html/bashdb.html) |
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

::: {.column}
| Linux |
|-----------------|
|[disk](html/fdisk.html)|
|[systemd](html/systemd.html)|
|[firewall](html/firewall.html) |
:::

::::::::::::::


:::::::::::::: {.columns}

::: {.column}
| Network |
|-----------------|
|[tcpdump](html/tcpdump.html) |
|[pcap-filter](html/pcap-filter.html) |
:::

::::::::::::::

