---
title: 'Alpine Single Node'
subtitle: '(Lab Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- The lab runs in a Docker container with a QEMU VM inside.
- The VM connects to the container through two network interfaces:
    - User-mode: -netdev user
    - Bridge: -netdev bridge,br=br0
- Static IP configured for SSH access:
    - Container: br0, 192.0.0.254
    - VM: eth1, 192.0.0.10
- Two virtual disks added to the VM:
    - disk_a.qcow2 (/dev/sda): Boot and filesystem.
    - disk_b.qcow2 (/dev/sdb): For testing.

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


# Setup
## Setup Container
- Clone code.
```sh
host$ git clone https://github.com/thachmpham/lab.git
host$ cd lab/alpine/1-node
```

- Setup the container.
```sh
host$ docker compose build
host$ docker compose up --detach
```


## Create VM
- Access the container.
```sh
host$ docker exec -it apk bash
```

- Create the VM.
```sh
cont$ ./create_vm.sh
```

- On the VM, create a file named ans with the same content as lab/alpine/1-node/ans.

- Install Alpine to the VM.
```sh
vm$ setup-alpine -e -f ans
```

- Shutdown the VM.
```sh
vm$ poweroff
```


## Setup SSH
- Start the VM.
```sh
cont$ ./start_vm.sh
```

- Allow root ssh login with a empty password.
```sh
vm$ vi /etc/ssh/sshd_config
# ------------------------------ #
PermitRootLogin yes
PasswordAuthentication yes
PermitEmptyPasswords yes
# ------------------------------ #

vm$ rc-service sshd restart
```

- Check ssh access from the container to the VM.
```sh
cont$ ssh vm
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