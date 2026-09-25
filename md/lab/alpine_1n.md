---
title: 'Alpine Single Node'
subtitle: '(Lab Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- Create a docker container with QEMU installed.
- Create a QEMU VM in the container.
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
192.0.0.254         192.0.0.10
   br0                  eth1
    +                    +
    +--------------------+


```

:::
::::::::::::::


# Create Container
```sh
host$ git clone https://github.com/thachmpham/lab.git
host$ cd lab/alpine/1-node
```

```sh
host$ docker compose build
host$ docker compose up --detach
host$ docker exec -it apk bash
```


# Create VM
```sh
container$ ./create_vm.sh
```

```sh
vm$ setup-alpine -e -f ans

vm$ vi /etc/ssh/sshd_config
PermitRootLogin yes
PasswordAuthentication yes
PermitEmptyPasswords yes

vm$ poweroff
```


# Start VM
```sh
container$ ./start_vm.sh
```


# SSH to VM
```sh
container$ ssh vm
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
- [QEMU Helper Networking](https://wiki.QEMU.org/Features/HelperNetworking)
- [Build a Security Test Lab](https://dev.to/zrouga/building-a-security-test-lab-with-QEMU-from-zero-to-network-monitoring-4onm)