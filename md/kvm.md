---
title:  'Kernel-based Virtual Machine'
---


## Usages

-----

### VM
```sh
  
virt-install --name node1 \
    --memory 4096 \
    --vcpus 2 \
    --disk path=/root/vm/node1.qcow2,size=50,format=qcow2 \
    --location 'https://debian.osuosl.org/debian/dists/stable/main/installer-amd64' \
    --nographics \
    --extra-args 'console=ttyS0,1152000n8 serial'
  
```

```sh
  
# show IP of VM
virsh domifaddr node1
  
```


### Snapshot
```sh
  
virsh snapshot-create-as node1 --name snapshot1
virsh snapshot-list node1
virsh snapshot-delete node1 --snapshotname snapshot1
  
```


### Network
```sh
  
virsh net-define mynetwork.xml
virsh net-start mynetwork
virsh net-autostart mynetwork
virsh net-list --all
virsh net-dumpxml mynetwork
  
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

<br>


## Procedures

-----

### Expand /root
```sh
  
qemu-img resize disk.qcow2 +10G
  
```
If /root mount to the last partition, enter the mantainance mode to [expand the partition](/html/fdisk.html).  
If /root not mount to the last partition, [create a new partition and mount /root to the new partition](/html/fdisk.html).