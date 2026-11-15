---
title: 'Alpine Two Nodes'
subtitle: '(Lab Setup Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column width=50%}

A lab to quickly setup 2 Alpine QEMU VMs.

- The VMs connect to each other through the below bridge interfaces.
    - br1 (191.0.0.0/24): For ssh.
    - br2 (192.0.0.0/24): For testing.
    - br3 (193.0.0.0/24): For testing.
- Each VM has the below disks.
   - /dev/sda: Boot and filesystem.
   - /dev/sdb: For testing.
   - /dev/sdc: For testing.

:::
::: {.column width=50%}

The VMs run within a docker container which acts as router, forwards traffics between VMs.
```go
         VM1            Container           VM2
          +                 +                +
      191.0.0.10 eth1 +----br1----+ eth1 191.0.0.20
      192.0.0.10 eth2 +----br2----+ eth2 192.0.0.20
      193.0.0.10 eth3 +----br3----+ eth3 193.0.0.20
          +                                  +
```

:::
::::::::::::::


# Setup
## Setup Container

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Clone code.
```sh
host$ git clone https://github.com/thachmpham/lab.git
```

- Step 2: Setup the container.
```sh
host$ cd lab/alpine/2-nodes
host$ docker compose build
host$ docker compose up --detach
```

:::
::: {.column width=50%}

- Step 3: Access the container.
```sh
host$ docker exec -it apk bash
```

:::
::::::::::::::


## Setup VM1

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Create vm1.
```sh
cont$ ./create_vm1.sh
```

- Step 2: On vm1, create a file named ans_vm1 with the same content as lab/alpine/2-node/ans_vm1.

- Step 3: Install Alpine to VM1.
```sh
vm1$ setup-alpine -e -f ans_vm1
```

- Step 4: Shutdown.
```sh
vm1$ poweroff
```

- Step 5: Start vm1.
```sh
cont$ ./start_vm1.sh
```

:::
::: {.column width=50%}

- Step 6: Allow root ssh login with a empty password.
```sh
vm1$ vi /etc/ssh/sshd_config
# ------------------------------ #
PermitRootLogin yes
PasswordAuthentication yes
PermitEmptyPasswords yes
# ------------------------------ #

vm1$ rc-service sshd restart
```

- Step 7: Check ssh access from the container to VM1.
```sh
cont$ ssh vm1
```

- Step 8: Enable community repository.
```sh
vm$ vi /etc/apk/repositories
http://dl-cdn.alpinelinux.org/alpine/v3.24/main
http://dl-cdn.alpinelinux.org/alpine/v3.24/community
```

:::
::::::::::::::


## Setup VM2

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Create vm2.
```sh
cont$ ./create_vm2.sh
```

- Step 2: On vm2, create a file named ans_vm2 with the same content as lab/alpine/2-node/ans_vm2.

- Step 3: Install Alpine to vm2.
```sh
vm2$ setup-alpine -e -f ans_vm2
```

- Step 4: Shutdown.
```sh
vm2$ poweroff
```

- Step 5: Start vm2.
```sh
cont$ ./start_vm2.sh
```

:::
::: {.column width=50%}

- Step 6: Allow root ssh login with a empty password.
```sh
vm2$ vi /etc/ssh/sshd_config
# ------------------------------ #
PermitRootLogin yes
PasswordAuthentication yes
PermitEmptyPasswords yes
# ------------------------------ #

vm2$ rc-service sshd restart
```

- Step 7: Check ssh access from the container to VM1.
```sh
cont$ ssh vm2
```

- Step 8: Enable community repository.
```sh
vm$ vi /etc/apk/repositories
http://dl-cdn.alpinelinux.org/alpine/v3.24/main
http://dl-cdn.alpinelinux.org/alpine/v3.24/community
```

:::
::::::::::::::


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