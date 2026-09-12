---
title: Alpine Single Node
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- Create a docker container with qemu.
- Create a qemu VM in the container.
- Connect container and VM through a bridge.
- Configure static IP.
- Allow SSH from container to VM.

:::
::: {.column}

```go
+------- Host ---------+
| +----- Container -+  |
| |   +- VM ---+    |  |
| |   |        |    |  |
| |   +--------+    |  |
| +-----------------+  |
+----------------------+
```

:::
::: {.column}

```go

Container               VM
192.0.0.2            192.0.0.10
   br0                  eth1
    +                    +
    +--------------------+


```

:::
::::::::::::::


# Setup
## Create Container
```sh
host$ git clone https://github.com/thachmpham/lab.git
host$ cd lab/alpine/use/single-node
```

```sh
host$ docker compose build
host$ docker compose up --detach
host$ docker exec -it apk bash
```


## Create VM
```sh
container$ ./create_vm.sh
```

```sh
vm$ setup-alpine
# - hostname:   vm
# - interface:  eth0
# - ipv4 addr:  dhcp
# - ipv6 addr:  auto
# - manual network config:  no
# - password:
# - timezone:   UTC
# - proxy:      none
# - ntp:        busybox
# - apk mirror: 1
# - setup user: no
# - ssh server: openssh
# - allow root ssh: yes
# - ssh key:    none
# - disk:       sda, sys

vm$ poweroff
```

```sh
container$ ./start_vm.sh
```


## Static IP
```sh
vm$ cat /etc/network/interfaces
# -------------------- #
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
hostname alpine-test

auto eth1
iface eth1 inet static
    address 192.0.0.10
    netmask 255.255.255.0
# -------------------- #
```

```sh
vm$ rc-service networking restart

vm$ ip addr show
eth1: inet 192.0.0.10
```


## Allow SSH
```sh
vm$ vi /etc/ssh/sshd_config
# -------------------- #
PermitRootLogin yes
# -------------------- #
```

```sh
vm$ rc-service sshd restart
```

```sh
container$ ssh root@192.0.0.10
```


# Shortcuts
QEMU keys:

- Ctrl + a h: Help Menu
- Ctrl + a x: Exit
- Ctrl + a c: Switch between VM console and QEMU monitor.


# References
- [Alpine Installation](https://wiki.alpinelinux.org/wiki/Installation)
- [Alpine QEMU](https://wiki.alpinelinux.org/wiki/QEMU)
- [Alpine SSH](https://wiki.alpinelinux.org/wiki/Setting_up_a_SSH_server)
- [QEMU Helper Networking](https://wiki.qemu.org/Features/HelperNetworking)
- [Build a Security Test Lab](https://dev.to/zrouga/building-a-security-test-lab-with-qemu-from-zero-to-network-monitoring-4onm)