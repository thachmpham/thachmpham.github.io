---
title: 'Fedora Single Node'
subtitle: '(Lab Series)'
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
   - /dev/sda: (Fedora Server QCOW) Boot and filesystem.
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
host$ cd lab/fedora/1-node
host$ docker compose build
host$ docker compose up --detach
```


## Create VM
- Access to container.
```sh
host$ docker exec -it f1 bash
```

- Create VM.
```sh
cont$ cd /ws
cont$ ./create_vm.sh
#
# Root password
#
```

- Reboot
```sh
vm$ reboot
```


## Setup VM
### Hostname
- Set hostname.
```sh
vm$ hostnamectl hostname vm
```


### Static IP
- Set static IP.
```sh
vm$ ip -br link show
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
ens3             UP             52:54:00:12:34:56 <BROADCAST,MULTICAST,UP,LOWER_UP>
ens4             UP             50:54:00:01:00:01 <BROADCAST,MULTICAST,UP,LOWER_UP>
ens5             UP             50:54:00:02:00:01 <BROADCAST,MULTICAST,UP,LOWER_UP>
ens6             UP             50:54:00:03:00:01 <BROADCAST,MULTICAST,UP,LOWER_UP>

vm$ nmcli conn show
NAME                UUID                                  TYPE      DEVICE
Wired connection 1  f1a11ae6-c99b-3dc9-a93e-b53d5c07b5e7  ethernet  ens3
Wired connection 2  9e51782f-a94e-36b3-a8b2-c7da6e005de7  ethernet  ens4
Wired connection 3  7fbc26d6-a1a0-388f-a13f-0e2e7d2cc7f2  ethernet  ens5
Wired connection 4  1967bc2c-709c-3b88-9fdb-babdc1c76902  ethernet  ens6
lo                  5eda62d0-1322-4a95-a564-a0a27a5ba757  loopback  lo

# set IP for ens4
vm$ nmcli conn modify 'Wired connection 2' ipv4.method manual ipv4.addr 191.0.0.10/24

# set IP for ens5
vm$ nmcli conn modify 'Wired connection 3' ipv4.method manual ipv4.addr 192.0.0.10/24

# set IP for ens6
vm$ nmcli conn modify 'Wired connection 4' ipv4.method manual ipv4.addr 193.0.0.10/24
```


### SSH
- Allow root login.
```sh
vm$ vi /etc/ssh/sshd_config
# ------------------------------ #
PermitRootLogin yes
PasswordAuthentication yes
# ------------------------------ #

vm$ systemctl restart sshd
```

- Check ssh.
```sh
cont$ ssh vm
```


# Shortcuts
QEMU keys:

- Ctrl + a h: Help Menu
- Ctrl + a x: Exit
- Ctrl + a c: Switch between VM console and QEMU monitor.

