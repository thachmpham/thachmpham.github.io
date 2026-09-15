---
title: Alpine Two Nodes
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- Create a docker container with qemu.
- Create 2 qemu VMs in the container.
- Connect container and VMs through a bridge.
- Configure static IP.
- Allow SSH from container to VMs.
- Allow ping between VMs.


:::
::: {.column}

```go
        VM1                 VM2
    192.0.0.10           192.0.0.20
       eth1                 eth1
        +                    +
        +--------------------+
                  +
                 br0
              192.0.0.254
               Container
```

:::
::::::::::::::


# Setup
## Setup Container
```sh
host$ git clone https://github.com/thachmpham/lab.git
host$ cd lab/alpine/2-nodes
```

```sh
host$ docker compose build
host$ docker compose up --detach
host$ docker exec -it apk bash
```


## Setup VM1
Create VM1.
```sh
container$ ./create_vm1.sh
```

```sh
vm1$ setup-alpine
# - hostname:   vm1
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

vm1$ poweroff
```

```sh
container$ ./start_vm1.sh
```

Configure static IP.
```sh
vm1$ cat /etc/network/interfaces
# -------------------- #
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
hostname alpine-test

auto eth1
iface eth1 inet static
    address 192.0.0.10/24
# -------------------- #
```

```sh
vm1$ rc-service networking restart

vm1$ ip addr show
eth1: inet 192.0.0.10
```


## Setup VM2
Create VM2.
```sh
container$ ./create_vm2.sh
```

```sh
vm2$ setup-alpine
# - hostname:   vm2
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

vm2$ poweroff
```

```sh
container$ ./start_vm2.sh
```

Configure static IP.
```sh
vm2$ cat /etc/network/interfaces
# -------------------- #
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
hostname alpine-test

auto eth1
iface eth1 inet static
    address 192.0.0.20/24
# -------------------- #
```

```sh
vm2$ rc-service networking restart

vm2$ ip addr show
eth1: inet 192.0.0.20
```


## Check
Check SSH.
```sh
container$ ssh root@192.0.0.10
container$ ssh root@192.0.0.20
```

Check ping.
```sh
# vm1 -> vm2
vm1$ ping 192.0.0.20
```

```sh
# vm2 -> vm1
vm2$ ping 192.0.0.10
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