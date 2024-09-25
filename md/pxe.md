---
title:  'Preboot eXecution Environment'
---


# 1. Introduction
Preboot eXecution Environment (PXE) is a protocol that allows computers to boot from a network, rather than from a local storage device like a hard drive or USB.  

PXE operates based on a client-server model.

- DHCP server assigns IP addresses and tells the client where to find the bootloader.
- TFTP server transfers the bootloader and necessary files to the client.
- Client requests an IP the DHCP server, downloads files via TFTP, and starts the boot process.


# 2. Labs
- Setup a PXE server with **dnsmasq**.
- Start a virtual machine that boot from the PXE server.

<script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
    mermaid.initialize({
        look: 'handDrawn',
        theme: 'neutral',
    });
</script>

<pre class="mermaid">
flowchart TD
    subgraph pxe-server[PXE Server: Host]
        dhcp-server[DHCP Server]
        tftp-server[TFTP Server]
    end

    client[PXE Client: Virtual Machine]
    
    pxe-server <--->|bridge: br0| client

</pre>


## 2.1. Setup PXE Server
Install `dmasq`.
```sh
  
$ apt install dnsmasq
  
```

Setup virtual bridge interface `br0`.
```sh
  
$ ip link add br0 type bridge
$ ip addr add 192.168.1.1/24 dev br0
$ ip link set br0 up
  
```

Edit file `/etc/dnsmasq.conf`.
```sh
  
interface=br0

dhcp-range=192.168.1.50,192.168.1.150,8h
dhcp-host=9e:42:87:f7:29:92,192.168.1.1,pc,infinite

dhcp-boot=pxelinux.0
enable-tftp
tftp-root=/srv/tftpboot
  
```

Create boot directory.
```sh
  
$ mkdir /srv/tftpboot
$ cd /srv/tftpboot
$ wget http://archive.ubuntu.com/ubuntu/dists/focal/main/installer-amd64/current/legacy-images/netboot/netboot.tar.gz
$ tar xvf netboot.tar.gz
$ rm -f netboot.tar.gz
  
```

Start `dnsmasq` service.
```sh
  
$ systemctl restart dnsmasq
  
```


## 2.2. Start a VM Boot from PXE.
Start a virtual machine that boot from the PXE server.
```sh
  
$ virt-install --name=vm1 \
    --ram=2048 \
    --disk size=16 \
    --pxe --boot network,menu=on \
    --network bridge=br0 \
    --graphic vnc \
    --os-variant=ubuntu20.04
  
```


## 2.4. Capture messages.
DHCP operates over UDP ports 67 (DHCP server), 68 (DHCP client) and 69 (TFTP server).
```sh
  
$ tshark -P -i br0 -f "port 67 or port 68 or port 69" -w pxe.pcap
  
```

- Output.
```sh
  
    1 0.000000000      0.0.0.0 → 255.255.255.255 DHCP 429 DHCP Discover - Transaction ID 0xd5db0a58
    2 1.007303971      0.0.0.0 → 255.255.255.255 DHCP 429 DHCP Discover - Transaction ID 0xd5db0a58
    3 3.007771996  192.168.1.1 → 192.168.1.132 DHCP 347 DHCP Offer    - Transaction ID 0xd5db0a58
    4 3.007901167  192.168.1.1 → 192.168.1.132 DHCP 347 DHCP Offer    - Transaction ID 0xd5db0a58
    5 3.013502444      0.0.0.0 → 255.255.255.255 DHCP 441 DHCP Request  - Transaction ID 0xd5db0a58
    6 3.016647551  192.168.1.1 → 192.168.1.132 DHCP 347 DHCP ACK      - Transaction ID 0xd5db0a58
    
    7 4.141973032 192.168.1.132 → 192.168.1.1  TFTP 82 Read Request, File: pxelinux.0, Transfer type: octet, blksize=1432, tsize=0
    8 4.303322778 192.168.1.132 → 192.168.1.1  TFTP 83 Read Request, File: ldlinux.c32, Transfer type: octet, tsize=0, blksize=1408
   ...
   19 4.450304505 192.168.1.132 → 192.168.1.1  TFTP 116 Read Request, File: ubuntu-installer/amd64/boot-screens/menu.cfg, Transfer type: octet, tsize=0, blksize=1408
   ...
   32 4.718978453 192.168.1.132 → 192.168.1.1  TFTP 92 Read Request, File: pxelinux.cfg/default, Transfer type: octet, tsize=0, blksize=1408
   33 4.721959936 192.168.1.132 → 192.168.1.1  TFTP 116 Read Request, File: ubuntu-installer/amd64/boot-screens/menu.cfg, Transfer type: octet, tsize=0, blksize=1408
   ...
   41 4.811254394 192.168.1.132 → 192.168.1.1  TFTP 118 Read Request, File: ubuntu-installer/amd64/boot-screens/splash.png, Transfer type: octet, tsize=0, blksize=1408
  
```


# References
- https://github.com/WillChamness/Dnsmasq-PXE
- https://autostatic.com/setting-up-a-pxe-server-with-dnsmasq
- https://ubuntu.com/server/docs/how-to-netboot-the-server-installer-on-amd64