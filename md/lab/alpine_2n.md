---
title: 'Alpine Two Nodes'
subtitle: '(Lab Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- Create a docker container with QEMU.
- Create 2 QEMU VMs in the container.
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
- Clone code.
```sh
host$ git clone https://github.com/thachmpham/lab.git
host$ cd lab/alpine/2-nodes
```

- Setup the container.
```sh
host$ docker compose build
host$ docker compose up --detach
```


## Setup VM1
- Access the container.
```sh
host$ docker exec -it apk bash
```

- Create VM1.
```sh
cont$ ./create_vm1.sh
```

- On VM1, create a file named ans_vm1 with the same content as lab/alpine/2-node/ans_vm1.

- Install Alpine to VM1.
```sh
vm1$ setup-alpine -e -f ans_vm1

vm1$ poweroff
```

- Start VM1.
```sh
cont$ ./start_vm1.sh
```

- Allow root ssh login with a empty password.
```sh
vm1$ vi /etc/ssh/sshd_config
# ------------------------------ #
PermitRootLogin yes
PasswordAuthentication yes
PermitEmptyPasswords yes
# ------------------------------ #

vm1$ rc-service sshd restart
```

- Check ssh access from the container to VM1.
```sh
cont$ ssh vm1
```


## Setup VM2
- Create VM2.
```sh
cont$ ./create_vm2.sh
```

- On VM2, create a file named ans_vm2 with the same content as lab/alpine/2-node/ans_vm2.

- Install Alpine to VM2.
```sh
vm2$ setup-alpine -e -f ans_vm2

vm2$ poweroff
```

- Start VM1.
```sh
cont$ ./start_vm2.sh
```

- Allow root ssh login with a empty password.
```sh
vm2$ vi /etc/ssh/sshd_config
# ------------------------------ #
PermitRootLogin yes
PasswordAuthentication yes
PermitEmptyPasswords yes
# ------------------------------ #

vm2$ rc-service sshd restart
```

- Check ssh access from the container to VM1.
```sh
cont$ ssh vm2
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