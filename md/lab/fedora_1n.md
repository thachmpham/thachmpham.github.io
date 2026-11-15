---
title: 'Fedora Single Node'
subtitle: '(Lab Setup Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column width=50%}

- The lab runs in a Docker container with a Fedora QEMU VM inside.
- The VM has 3 network interfaces and IP as below.
    - br1 (191.0.0.0/24): For ssh.
    - br2 (192.0.0.0/24): For testing.
    - br3 (193.0.0.0/24): For testing.
- Each VM has the below disks.
   - /dev/sda: (Fedora Server QCOW) Boot and filesystem.
   - /dev/sdb: For testing.
   - /dev/sdc: For testing.

:::
::: {.column width=50%}

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

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Clone code.
```sh
host$ git clone https://github.com/thachmpham/lab.git
```

- Step 2: Setup container.
```sh
host$ cd lab/fedora/1-node
host$ docker compose build
host$ docker compose up --detach
```

:::
::: {.column width=50%}

- Step 3: Access to container.
```sh
host$ docker exec -it f1 bash
```

:::
::::::::::::::


## Create VM

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Create VM.
```sh
cont$ cd /ws
cont$ ./create_vm.sh
# Remember to set root password
```

:::
::: {.column width=50%}

- Step 2: Reboot.
```sh
vm$ reboot
```

:::
::::::::::::::


## Setup VM
### Hostname
- Set hostname.
```sh
vm$ hostnamectl hostname vm
```


### Static IP

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Show connections and interfaces.
```sh
vm$ nmcli conn show
NAME                UUID                                  TYPE      DEVICE
Wired connection 1  f1a11ae6-c99b-3dc9-a93e-b53d5c07b5e7  ethernet  ens3
Wired connection 2  9e51782f-a94e-36b3-a8b2-c7da6e005de7  ethernet  ens4
Wired connection 3  7fbc26d6-a1a0-388f-a13f-0e2e7d2cc7f2  ethernet  ens5
Wired connection 4  1967bc2c-709c-3b88-9fdb-babdc1c76902  ethernet  ens6
```

:::
::: {.column width=50%}

- Step 2: Set static IPs.
```sh
# set IP for ens4
vm$ nmcli conn modify 'Wired connection 2' ipv4.method manual ipv4.addr 191.0.0.10/24

# set IP for ens5
vm$ nmcli conn modify 'Wired connection 3' ipv4.method manual ipv4.addr 192.0.0.10/24

# set IP for ens6
vm$ nmcli conn modify 'Wired connection 4' ipv4.method manual ipv4.addr 193.0.0.10/24
```

:::
::::::::::::::

### SSH

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Allow root login.
```sh
vm$ vi /etc/ssh/sshd_config
# ------------------------------ #
PermitRootLogin yes
PasswordAuthentication yes
# ------------------------------ #

vm$ systemctl restart sshd
```

:::
::: {.column width=50%}

- Step 2: Check ssh.
```sh
cont$ ssh vm
```

:::
::::::::::::::

# Shortcuts
QEMU keys:

- Ctrl + a h: Help Menu
- Ctrl + a x: Exit
- Ctrl + a c: Switch between VM console and QEMU monitor.

