---
title:  'Kernel-based Virtual Machine'
---

## 1. Create VM
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

## 3. Inspect VM
- Get IPs of a VM.
```sh
  
$ virsh domifaddr node1
  
```

## References
- https://linux.die.net/man/1/virt-install
- https://www.libvirt.org/manpages/virsh.html