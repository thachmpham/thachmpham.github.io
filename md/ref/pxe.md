---
title: "Preboot eXecution Environment"
---


## Lab: PXE Ubuntu Netboot
### Server

:::::::::::::: {.columns}
::: {.column width=50%}

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

Setup boot directory.
```sh
$ mkdir /var/ftpd
$ cd /var/ftpd

$ wget http://archive.ubuntu.com/ubuntu/dists/bionic-updates/main/installer-amd64/current/images/netboot/netboot.tar.gz
$ tar xf netboot.tar.gz
```

:::
::: {.column width=50%}

**Workaround**: Install Ubuntu over serial console.

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

<br>

### Client
Start VM with PXE.
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

## Lab: PXE NFS Root Filesystem
### Server
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

Edit /etc/exports.
```sh
/var/nfs *(rw, sync, no_subtree_check, no_root_squash)
```

Export nfs directory.
```sh
$ export -a
$ export -v
```

## References
- [PXELINUX wiki](https://wiki.syslinux.org/wiki/index.php?title=PXELINUX).
- [Ubuntu network installer](https://ubuntu.com/download/alternative-downloads#network-installer).
- [Ubuntu netboot images](https://cdimage.ubuntu.com/netboot/).