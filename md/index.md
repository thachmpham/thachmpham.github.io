---
title:  "Thach Pham's Notebook"
subtitle: "A Software Engineer's Practical Notebook<br/>*Computer Systems - Distributed Systems - Software Debugging*"
---


## 1. Computer Systems
### 1.1. Virtual Network Interfaces
| |
|:-----------------|----------------:|
| [Virtual Ethernet Interface](html/veth.html) | (*ip veth netns*) |
| [Virtual Bridge Interface](html/vbridge.html) | *(ip bridge netns)* |
| [Dummy Interface](html/vdummy-interface.html) | *(ip dummy netns)* |
| [Bond Interface](html/vbond-interface.html) | *(ip bond netns)* |


### 1.2. Networking
| |
|:-----------------|----------------:|
| [Address Resolution Protocol](html/arp.html) | *(arp, arping)* |
| [IP Routing](html/ip-routing.html) | *(ip route)* |
| [Dynamic Host Configuration Protocol](html/dhcp.html) | *(kea-dhcp4)* |
| [TIPC Protocol](html/tipc.html) | *(tipc)* |
| [Virtual IP Address](html/vip.html) | *(keepalived)* |


### 1.3. Infrastructure
| |
|:-----------------|----------------:|
| [Preboot eXecution Environment](html/pxe.html) | *(dnsmasq)* |
| [Lightweight Directory Access Protocol](html/ldap.html) | *(openldap)* |
| [Distributed Replicated Block Device](html/drbd.html) | *(drbd)* |


### 1.4. Linux
| |
|:-----------------|----------------:|
| [Kernel Virtual Machine](html/kvm.html)           | *(kvm)*  |
| [Disk Manipulation](html/fdisk.html)   | *(fdisk)*  |
| [SystemD Management](html/systemd.html)           | *(systemd)*  |

<br>


## 2. Distributed Systems
### 2.1. OpenSAF
| |
|:-----------------|----------------:|
| [Build & Install](html/opensaf-install.html) | *(make)* |
| [Setup Cluster &nbsp;&nbsp; - &nbsp; 1 SC](html/opensaf-1sc.html) | *(opensaf cluster)* |
| [Setup Cluster &nbsp;&nbsp; - &nbsp; 2 SCs](html/opensaf-2sc.html) | *(opensaf cluster)* |
| [Information Model Management](html/opensaf-imm.html) | *(opensaf imm)* |
| [Software Management Framework](html/opensaf-smf.html) | *(opensaf smf)* |
| [Log Service](html/opensaf-log.html) | *(opensaf log)* |
| [Convert Program to High Availability](html/opensaf-amf-non-sa-aware.html) | *(opensaf amf)* |
| [Build High Availability Application](html/opensaf-amf-sa-aware.html) | *(opensaf amf)* |
| [Healthcheck](html/opensaf-healthcheck.html) | *(opensaf amf)* |

<br>


## 3. Software Debugging
### 3.1 Debug C/C++ with GDB
| |
|:-----------------|----------------:|
| [Debug a Program Started by a Script](html/gdb-program-started-by-script.html)    | *(gdb catch)*     |
| [Debug A Program Started by SystemD](html/gdb-program-started-by-systemd.html)    | *(gdb remote)*    |
| [Debug a High Availability Program](html/gdb-ha-program.html) | *(gdb non-stop)* |
| [Search Memory](html/gdb-find.html)                   | *(gdb find)*  |
| [Dump & Restore Memory](html/gdb-dump-restore.html)   | *(gdb dump restore)*  |
| [Execute Functions](html/gdb-call.html)               | *(gdb call)*  |
| [Inspect ELF](html/elf.html)                    | *(gdb objdump)*  |
| [Virtual Address & EFL Offset](html/virtual-addr-elf-offset.html)        | *(gdb sharedlibrary)*   |

### 3.2. Extend GDB with Python
| |
|:-----------------|----------------:|
| [Print STL Containers](html/gdb-stl.html)             | *(gdb pretty-printer)*   |
| [Print Complex Data](html/gdb-write-pp.html)             | *(gdb pretty-printer)*   |


### 3.3. Binary Analysis
| |
|:-----------------|----------------:|
| [Inside the Call Stack](html/asm-callstack.html) | *(gcc objdump)* |


### 3.4. Debug Bash Scripts
| |
|:-----------------|----------------:|
| [Step Through Bash Script](html/bashdb.html) | *(bashdb rpmbuild)* |

<br>


## 5. Tools
### 5.1. Diagram
| |
|:-----------------|----------------:|
| [PlantUML](html/plantuml.html) | *(uml)* |
| [Graphviz](html/graphviz.html) | *(dot)* |

