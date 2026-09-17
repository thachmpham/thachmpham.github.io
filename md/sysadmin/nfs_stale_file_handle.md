---
title: 'Reproduce NFS Stale File Handle'
---


# Overview
- Setup an NFS lab.
- Reproduce the 'Stale File Handle' error:
    - Scenario 1: Delete while open.

# Lab
- Setup lab: [Network File System (NFS)](/html/lab/nfs.html).
- VM1 acts as NFS server.
```sh
vm1$ exportfs -v
/srv/nfs 192.0.0.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)

vm1$ showmount -a
192.0.0.10:/srv/nfs
192.0.0.20:/srv/nfs
```

- VM2 acts as NFS client.
```sh
vm2$ df -h
Filesystem                Size      Used Available Use% Mounted on
192.0.0.10:/srv/nfs       5.6G    221.0M      5.1G   4% /mnt/nfs
```


# Reproduce Stale File 

## Delete while Open

If the server deletes a file or directory while the client currently has open, the inode held by client become invalid.
The further actions to the inode will fail with 'Stale file handle' error.

:::::::::::::: {.columns}
::: {.column width=50%}

**Server**. 

- Step 1: Create a directory.
```sh
vm1$ mkdir /srv/nfs/test
```

- Step 3: Delete the directory.
```sh
vm1$ rm -rf /srv/nfs/test
```

:::
::: {.column width=50%}

**Client**. 

- Step 2: Go to the directory.
```sh
vm2$ cd /mnt/nfs/test
```

- Step 4: List files.
```sh
vm2:/mnt/nfs/test$ ls
ls: .: Stale file handle
```

:::
::::::::::::::
