---
title:  "Thach Pham's Notebook"
subtitle: "A Software Engineer's Practical Notebook<br/>*Computer Systems - Distributed Systems - Software Debugging*"
---


## Computer Systems
### Virtual Network Interfaces
| |
|:-----------------|----------------:|
| [Virtual Ethernet Interface](html/veth.html) | (*ip link veth*) |
| [Virtual Bridge Interface](html/vbridge.html) | *(ip link bridge)* |
| [Dummy Interface](html/vdummy-interface.html) | *(ip link dummy)* |
| [Bond Interface](html/vbond-interface.html) | *(ip link bond)* |


### TCP/IP
| |
|:-----------------|----------------:|
| [Address Resolution Protocol](html/arp.html) | *(arp, arping)* |
| [IP Routing](html/ip-routing.html) | *(ip route, ping)* |
| [Dynamic Host Configuration Protocol](html/dhcp.html) | *(kea-dhcp4, dhclient)* |

### Infrastructure
| |
|:-----------------|----------------:|
| [Preboot eXecution Environment](html/pxe.html) | *(dnsmasq, kvm)* |
| [Lightweight Directory Access Protocol](html/ldap.html) | *(openldap)* |
| [Virtual IP Address](html/vip.html) | *(keepalived, docker)* |
| [Distributed Replicated Block Device](html/drbd.html) | *(drbd)* |


### Linux
| |
|:-----------------|----------------:|
| [Kernel Virtual Machine](html/kvm.html)           | *(kvm, qemu)*  |
| [Disk Manipulation](html/fdisk.html)   | *(fdisk)*  |
| [SystemD Management](html/systemd.html)           | *(systemd)*  |

<br>


## Distributed Systems
### OpenSAF
| |
|:-----------------|----------------:|
| [Build & Install](html/opensaf-install.html) | *(docker, make)* |
| [Setup Cluster &nbsp;&nbsp; - &nbsp; 1 SC](html/opensaf-1sc.html) | *(opensaf cluster)* |
| [Setup Cluster &nbsp;&nbsp; - &nbsp; 2 SCs](html/opensaf-2sc.html) | *(opensaf cluster)* |
| [Information Model Management](html/opensaf-imm.html) | *(opensaf imm)* |
| [Software Management Framework](html/opensaf-smf.html) | *(opensaf smf)* |
| [Log Service](html/opensaf-log.html) | *(opensaf log)* |
| [Convert Program to High Availability](html/opensaf-amf-non-sa-aware.html) | *(opensaf amf)* |
| [Build High Availability Application](html/opensaf-amf-sa-aware.html) | *(opensaf amf)* |
| [Healthcheck](html/opensaf-healthcheck.html) | *(opensaf amf)* |

<br>


## Software Debugging
### Debug C/C++
| |
|:-----------------|----------------:|
| [Debug a Program Started by a Script](html/gdb-program-started-by-script.html)    | *(gdb catch)*     |
| [Debug A Program Started by SystemD](html/gdb-program-started-by-systemd.html)    | *(gdb remote)*    |
| [Debug a High Availability Program](html/gdb-ha-program.html) | *(gdb non-stop)* |
| [Search Memory](html/gdb-find.html)                   | *(gdb find)*  |
| [Dump & Restore Memory](html/gdb-dump-restore.html)   | *(gdb dump restore)*  |
| [Execute Functions](html/gdb-call.html)               | *(gdb call)*  |
| [Print STL Containers](html/gdb-stl.html)             | *(gdb info pretty-printer)*   |
| [Inspect ELF Files](html/elf.html)                    | *(objdump section-headers syms)*  |
| [Shared Library](html/cpp-shared-library.html)        | *(gdb info sharedlibrary)*   |


### From C to x86 Assembly
| |
|:-----------------|----------------:|
| [Call stack](html/c-callstack.html) | *(gdb tui layout asm regs)* |


### Debug Bash
| |
|:-----------------|----------------:|
| [BashDB](html/bashdb.html) | *(bashdb, rpmbuild)* |

<br>


## Tools
### Diagram
| |
|:-----------------|----------------:|
| [PlantUML](html/plantuml.html) | *(uml)* |
| [Graphviz](html/graphviz.html) | *(dot)* |

