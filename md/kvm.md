---
title:  'Kernel-based Virtual Machine'
---

# 1. VM Management
## 1.1 Create VM
```sh
  
# create Debian VM: net-installer, nographics.
$ virt-install --name node1 \
    --memory 4096 \
    --vcpus 2 \
    --disk path=/root/vm/node1.qcow2,size=50,format=qcow2 \
    --location 'https://debian.osuosl.org/debian/dists/stable/main/installer-amd64' \
    --nographics \
    --extra-args 'console=ttyS0,1152000n8 serial'
  
```

- [linux.die.net/man/1/virt-install](https://linux.die.net/man/1/virt-install)


## 1.2. Inspect VM
```sh
  
# show IP of VM
$ virsh domifaddr node1
  
```

- [www.libvirt.org/manpages/virsh.html](https://www.libvirt.org/manpages/virsh.html)


# 2. Snapshot
```sh
  
$ virsh snapshot-create-as node1 --name snapshot1

$ virsh snapshot-list node1

$ virsh snapshot-delete node1 --snapshotname snapshot1
  
```


# 3. Network
```sh
  
$ virsh net-define mynetwork.xml

$ virsh net-start mynetwork

$ virsh net-autostart mynetwork

$ virsh net-list --all

$ virsh net-dumpxml mynetwork
  
```

```xml
  
<network>
    <name>mynetwork</name>
    <forward mode="nat"/>
    <ip address="192.168.122.1" netmask="255.255.255.0">
        <dhcp>
            <range start="192.168.122.2" end="192.168.122.254"/>
        </dhcp>
    </ip>
</network>
  
```

- [libvirt.org/formatnetwork.html](https://libvirt.org/formatnetwork.html)


# 4. Disk
## 4.1. Expand /root Partition
- Expand disk image.
```sh
  
$ qemu-img resize disk.qcow2 +10G
  
```

- If /root mount to the last partition, enter the mantainance mode to [expand the partition](/html/fdisk.html).
- If /root mount to the last partition, [create a new partition and mount /root to the new partition](/html/fdisk.html).