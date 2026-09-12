---
title: PXE Boot
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- PXE server is a docker container. Provides:
    - DHCP server (isc-dhcp-server).
        - Allocate IP for PXE clients.
        - Specify IP of boot server (next-server), who store the boot files.
        - Specify the boot file (pxelinux.0).
    - TFTP server (atftpd).
        - Host the directory that contains boot files.
- PXE client is a qemu VM, boot from network.
- PXE server and client connect through a bridge.

:::
::: {.column}

```go


PXE Server            PXE Client
(Container)             (VM)
192.0.0.2            192.0.0.1XX
   br0                  eth1
    +                    +
    +--------------------+


```

:::
::::::::::::::


# PXE Server
```sh
host$ git clone https://github.com/thachmpham/lab.git
host$ cd lab/pxe/use/setup-02
```

```sh
host$ docker compose build
host$ docker compose up
host$ docker exec -it pxe bash
```


## DHCP Server
The DHCP server automatically start together the container startup.
```sh
container$ ps ax
  PID TTY      STAT   TIME COMMAND
   46 ?        Ss     0:00 /usr/sbin/dhcpd -4 -q -cf /etc/dhcp/dhcpd.conf br0
```


Listening at port 67.
```sh
container$ lsof -P -i -n
COMMAND PID   USER FD   TYPE DEVICE SIZE/OFF NODE NAME
dhcpd    46   root 8u  IPv4   9300      0t0  UDP *:67
```


Show DHCP configuration.
```sh
container$ cat /etc/dhcp/dhcpd.conf
# ---------------------------------------- #
allow booting;
allow bootp;

subnet 192.0.0.0 netmask 255.255.255.0 {
    range 192.0.0.100 192.0.0.150;          # range for client address
    option broadcast-address 192.0.0.255;

    next-server 192.0.0.2;                  # where is boot file
    option subnet-mask 255.255.255.0;
    filename "/pxelinux.0";                 # boot file
}
# ---------------------------------------- #
```


```sh
pxe$ cat /etc/default/isc-dhcp-server
# ---------------------------------------- #
# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#   Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACESv4="br0"
# ---------------------------------------- #
```


## TFTP Server
The TFTP server automatically start together the container startup.
```sh
pxe$ ps ax | grep -P 'dhcp|tftp'
  PID TTY      STAT   TIME COMMAND
   59 ?        Ss     0:00 /usr/sbin/atftpd --daemon --port 69 --verbose=7 /srv/tftp
```

Listening at port 69.
```sh
pxe$ lsof -P -i -n
COMMAND PID   USER FD   TYPE DEVICE SIZE/OFF NODE NAME
atftpd   59 nobody 0u  IPv6   8401      0t0  UDP *:69
```

Show TFTP configuration.
```sh
container$ cat /etc/default/atftpd
# ---------------------------------------- #
OPTIONS="--port 69 --verbose=7 /srv/tftp"
# ---------------------------------------- #
```

TFTP server hosts the boot directory (/srv/tftp).
```sh
container$ tree /srv/tftp
.
|-- ldlinux.c32     # control module: display menu.
|-- pxelinux.0      # bootloader binary: start boot process in client.
|-- pxelinux.cfg    # boot menu.
|-- ubuntu-installer
|   `-- amd64
|       |-- boot-screens
|       |   |-- txt.cfg         # menu when in text graphics.
|       |-- initrd.gz           # ramdisk, tools to load filesystem.
|       |-- linux               # kernel.
```

```sh
container$ cat /srv/tftp/ubuntu-installer/amd64/boot-screens/txt.cfg
# ---------------------------------------- #
default install
label install
        menu label ^Install
        menu default
        kernel ubuntu-installer/amd64/linux
        append vga=788 initrd=ubuntu-installer/amd64/initrd.gz --- console=ttyS0,19200 earlyprint=serial,ttyS0,19200
label cli
        menu label ^Command-line install
        kernel ubuntu-installer/amd64/linux
        append tasks=standard pkgsel/language-pack-patterns= pkgsel/install-language-support=false vga=788 initrd=ubuntu-installer/amd64/initrd.gz --- console=ttyS0,19200 earlyprint=serial,ttyS0,19200
# ---------------------------------------- #
```


# PXE Client
Start VM, boot from network.
```sh
container$ ./start_vm.sh
```


# References
- [Ubuntu Setup Netboot Installer Server](https://ubuntu.com/server/docs/how-to/installation/how-to-netboot-the-server-installer-on-amd64/).
- [Ubuntu Setup ATFTP](https://unixwars.blogspot.com/2016/09/setting-up-atftp-in-ubuntu-linux.html).