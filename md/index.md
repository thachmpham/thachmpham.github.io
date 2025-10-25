---
title:  "Thach Pham's Notebook"
subtitle: "A Software Engineer's Notebook<br/>*Computer Systems - Distributed Systems - Software Debugging*"
---


## 1. Networking
### 1.1. Interfaces
| |
|:-----------------|----------------:|
| [Virtual Ethernet Interface](html/veth.html) | (*ip veth netns*) |
| [Virtual Bridge Interface](html/vbridge.html) | *(ip bridge netns)* |
| [Dummy Interface](html/vdummy-interface.html) | *(ip dummy netns)* |
| [Bond Interface](html/vbond-interface.html) | *(ip bond netns)* |


### 1.2. Protocols
| |
|:-----------------|----------------:|
| [Address Resolution Protocol](html/arp.html) | *(arp, arping)* |
| [IP Routing](html/ip-routing.html) | *(ip route)* |
| [Dynamic Host Configuration Protocol](html/dhcp.html) | *(kea-dhcp4)* |
| [Virtual IP Address](html/vip.html) | *(keepalived)* |
| [Preboot eXecution Environment](html/pxe.html) | *(dnsmasq)* |
| [Lightweight Directory Access Protocol](html/ldap.html) | *(openldap)* |
| [Distributed Replicated Block Device](html/drbd.html) | *(drbd)* |
| [Transparent Inter-Process Communication](html/tipc.html) | *(tipc)* |

<br>


## 2. Distributed Systems
### 2.1. OpenSAF
| |
|:-----------------|----------------:|
| [Build & Install](html/opensaf-install.html) | *(make)* |
| [Setup Cluster &nbsp;&nbsp; - &nbsp; 1 SC](html/opensaf-1sc.html) | *(opensaf docker)* |
| [Setup Cluster &nbsp;&nbsp; - &nbsp; 2 SCs](html/opensaf-2sc.html) | *(opensaf docker)* |
| [Information Model Management](html/opensaf-imm.html) | *(imm)* |
| [Software Management Framework](html/opensaf-smf.html) | *(smf)* |
| [Log Service](html/opensaf-log.html) | *(log)* |
| [Convert Program to High Availability](html/opensaf-amf-non-sa-aware.html) | *(amf non sa-aware)* |
| [Build High Availability Application](html/opensaf-amf-sa-aware.html) | *(amf sa-aware)* |
| [Healthcheck](html/opensaf-healthcheck.html) | *(amf)* |
| [Error Detection & Recovery](html/opensaf-amf-error-detection.html) | *(amf component)* |

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
:::

::::::::::::::

