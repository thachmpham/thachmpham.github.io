---
title:  'Kernel-based Virtual Machine'
---

## 1. VM
- Create Debian VM using net-installer and nographics.
```sh
  
$ virt-install --name node1 \
    --memory 4096 \
    --vcpus 2 \
    --disk path=/root/vm/node1.qcow2,size=50,format=qcow2 \
    --location 'https://debian.osuosl.org/debian/dists/stable/main/installer-amd64' \
    --nographics \
    --extra-args 'console=ttyS0,1152000n8 serial'
  
```

- Get IPs of a VM.
```sh
  
$ virsh domifaddr node1
  
```

## 2. Snapshot
- Create a snapshot.
```sh
  
$ virsh snapshot-create-as node1 --name snapshot1
  
```

- List snapshots.
```sh
  
$ virsh snapshot-list node1
  
```

- Delete a snapshot.
```sh
  
$ virsh snapshot-delete node1 --snapshotname snapshot1
  
```

## 3. Network
- A sample network config.
```xml
  
<network>
    <name>network_1</name>
    <forward mode="nat"/>
    <ip address="192.168.122.1" netmask="255.255.255.0">
        <dhcp>
            <range start="192.168.122.2" end="192.168.122.254"/>
        </dhcp>
    </ip>
</network>
  
```

- Create a network.
```sh
  
$ virsh net-define network_1.xml
  
```

- Start a network.
```sh
  
$ virsh net-start network_1
  
```

- Enable autostart.
```sh
  
$ virsh net-autostart network_1
  
```

- Show info.
```sh
  
$ virsh net-list --all

$ virsh net-dumpxml network_1
  
```


## References
- https://linux.die.net/man/1/virt-install
- https://www.libvirt.org/manpages/virsh.html
- https://libvirt.org/formatnetwork.html