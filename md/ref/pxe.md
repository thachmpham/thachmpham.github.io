---
title: "Preboot eXecution Environment"
---


## Lab 01: PXE Ubuntu Netboot

:::::::::::::: {.columns}
::: {.column width=50%}

Server:

- PXE server provides Ubuntu Netboot.

Client:

- Boot from PXE.
- Install Ubuntu to hard disk.

### 1. Server

Install dnsmasq
```sh
$ apt install dnsmasq
```

Setup interface.
```sh
$ ip link add br0 type bridge
$ ip addr add 192.168.0.1/24 dev br0
$ ip link set br0 up
```

Edit /etc/dnsmasq.conf
```sh
interface=br0
dhcp-range=192.168.0.50,192.168.0.150,12h
dhcp-boot=pxelinux.0
enable-tftp
tftp-root=/var/ftpd
```

Start server.
```sh
$ systemctl start dnsmasq
```

:::
::: {.column width=50%}

Setup boot directory.
```sh
$ mkdir /var/ftpd
$ cd /var/ftpd

$ wget http://archive.ubuntu.com/ubuntu/dists/bionic-updates/main/installer-amd64/current/images/netboot/netboot.tar.gz
$ tar xf netboot.tar.gz
```

**Workaround**: Enable serial console.

Prepend to /var/ftpd/pxelinux.cfg/default
```sh
console 0
serial 0 19200 0
```

Edit /var/ftpd/ubuntu-installer/amd64/boot-screens/txt.cfg.
```sh
label install
	menu label ^Install
	menu default
	kernel ubuntu-installer/amd64/linux
    # append vga=788 initrd=ubuntu-installer/amd64/initrd.gz --- quiet
    append initrd=ubuntu-installer/amd64/initrd.gz --- console=ttyS0,19200 earlyprint=serial,ttyS0,19200 	
```

### 2. Client

```sh
$ virt-install --name=vm1 \
    --ram=2048 \
    --disk size=16 \
    --pxe --boot network,menu=on \
    --network bridge=br0 \
    --os-variant ubuntu18.04 \
    --nographic
```

:::
::::::::::::::

* * * * * 

<br>


## Lab 02: NFS Root Filesystem

:::::::::::::: {.columns}
::: {.column width=50%}

Continue of lab 01.

Server:

- PXE server provides linux kernel.
- Provide NFS root filesystem.

Client

- Boot from PXE.

### 1. Server

Copy linux kernel and initrd to ftpd directory.
```sh
$ cp vmlinuz-5.15.0-164-generic /var/ftpd
$ cp initrd.img-5.15.0-164-generic /var/ftpd
```

Edit /var/ftpd/ubuntu-installer/amd64/boot-screens/txt.cfg
```sh
label install
	menu label ^Install
	menu default
	kernel vmlinuz-5.15.0-164-generic
    append initrd=initrd.img-5.15.0-164-generic root=/dev/nfs nfsroot=192.168.0.1:/var/nfs --- console=ttyS0,19200 earlyprint=serial,ttyS0,19200
```

:::
::: {.column width=50%}

Prepare root filesystem.

- Built root filesystem with [builtroot](https://buildroot.org/).
- Mount and copy all files in root filesystem to /var/nfs.

Setup NFS server.

Edit /etc/exports.
```sh
/var/nfs *(rw, sync, no_subtree_check, no_root_squash)
```

Export nfs directory.
```sh
$ export -a
$ export -v
```

### 2. Client

```sh
$ virt-install --name=vm1 \
    --ram=2048 \
    --disk size=16 \
    --pxe --boot network,menu=on \
    --network bridge=br0 \
    --os-variant ubuntu18.04 \
    --nographic
```

:::
::::::::::::::

* * * * *

<br>


## Boot Process

Server Log.
```sh
$ journalctl -f -u dnsmasq
```

```sh
Jan 23 08:02:42 pc dnsmasq-dhcp[5179]: DHCPDISCOVER(br0) 52:54:00:63:77:ac
Jan 23 08:02:42 pc dnsmasq-dhcp[5179]: DHCPOFFER(br0) 192.168.0.50 52:54:00:63:77:ac
Jan 23 08:02:45 pc dnsmasq-dhcp[5179]: DHCPREQUEST(br0) 192.168.0.50 52:54:00:63:77:ac
Jan 23 08:02:45 pc dnsmasq-dhcp[5179]: DHCPACK(br0) 192.168.0.50 52:54:00:63:77:ac

Jan 23 08:02:46 pc dnsmasq-tftp[5179]: sent /var/ftpd/pxelinux.0 to 192.168.0.50
Jan 23 08:02:46 pc dnsmasq-tftp[5179]: sent /var/ftpd/ldlinux.c32 to 192.168.0.50

Jan 23 08:02:46 pc dnsmasq-tftp[5179]: sent /var/ftpd/pxelinux.cfg/default to 192.168.0.50
Jan 23 08:02:47 pc dnsmasq-tftp[5179]: sent /var/ftpd/ubuntu-installer/amd64/boot-screens/stdmenu.cfg to 192.168.0.50
Jan 23 08:02:47 pc dnsmasq-tftp[5179]: sent /var/ftpd/ubuntu-installer/amd64/boot-screens/txt.cfg to 192.168.0.50
Jan 23 08:02:47 pc dnsmasq-tftp[5179]: sent /var/ftpd/ubuntu-installer/amd64/boot-screens/stdmenu.cfg to 192.168.0.50
Jan 23 08:02:47 pc dnsmasq-tftp[5179]: sent /var/ftpd/ubuntu-installer/amd64/boot-screens/adtxt.cfg to 192.168.0.50
Jan 23 08:02:47 pc dnsmasq-tftp[5179]: sent /var/ftpd/ubuntu-installer/amd64/boot-screens/rqtxt.cfg to 192.168.0.50

Jan 23 08:03:00 pc dnsmasq-tftp[5179]: sent /var/ftpd/vmlinuz-5.15.0-164-generic to 192.168.0.50
Jan 23 08:04:59 pc dnsmasq-tftp[5179]: sent /var/ftpd/initrd.img-5.15.0-164-generic to 192.168.0.50
```

:::::::::::::: {.columns}
::: {.column width=60%}

| File | Usage          |
|:-----|:---------------|
| pxelinux.0            | PXE bootloader. Loads config, kernel, initrd.|
| pxelinux.cfg/default  | Configuration and boot menu.  |
| vmlinuz               | Linux kernel.  |
| initrd.img            | Initial RAM disk. Temporary rootfs in RAM to load drivers and mount real rootfs. |

:::
::: {.column width=40%}

<pre class="mermaid">
sequenceDiagram
    Client ->> Server: DHCPDISCOVER
    Server ->> Client: DHCPOFFER    
    Client ->> Server: DHCPREQUEST
    Server ->> Client: DHCPACK
    Server ->> Client: pxelinux.0
    Server ->> Client: pxelinux.cfg/default
    Server ->> Client: vmlinuz
    Server ->> Client: initrd.img
</pre>

:::
::::::::::::::

* * * * *

<br>

## References
- [PXELINUX](https://wiki.syslinux.org/wiki/index.php?title=PXELINUX).
- [Ubuntu network installer](https://ubuntu.com/download/alternative-downloads#network-installer).
- [Ubuntu netboot images](https://cdimage.ubuntu.com/netboot/).
- [NFS Root Filesystem](https://www.kernel.org/doc/Documentation/filesystems/nfs/nfsroot.txt).