---
title:  'Kernel-based Virtual Machine'
---

# 1. Create Virtual Machines
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


# References
- https://linux.die.net/man/1/virt-install
- https://www.libvirt.org/manpages/virsh.html