---
title: 'Alpine Single Node'
subtitle: '(Lab Setup Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- The lab runs in a Docker container with two QEMU VMs inside.
- The VM has three interfaces:
    - br1 (191.0.0.0/24): For ssh.
    - br2 (192.0.0.0/24): For testing.
    - br3 (193.0.0.0/24): For testing.
- Each VM has the below disks:
   - /dev/sda: Boot and filesystem.
   - /dev/sdb: For testing.
   - /dev/sdc: For testing.

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
::::::::::::::


# Setup
## Setup Container
- Clone code.
```sh
host$ git clone https://github.com/thachmpham/lab.git
```

- Setup container.
```sh
host$ cd lab/alpine/1-node
host$ docker compose build
host$ docker compose up --detach
```


## Create VM
- Access to container.
```sh
host$ docker exec -it apk bash
```

- Create the VM.
```sh
cont$ cd /ws
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

- Check ssh from container to vm.
```sh
cont$ ssh vm
```


# Setup Repository
- Enable community repository.
```sh
vm$ vi /etc/apk/repositories
http://dl-cdn.alpinelinux.org/alpine/v3.24/main
http://dl-cdn.alpinelinux.org/alpine/v3.24/community
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