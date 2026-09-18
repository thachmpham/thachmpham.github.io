---
title: 'NFS Stale File Handle'
---


# Overview
When an NFS client accesses an inode that no longer available on the server, the stale file handle error (ESTALE) occurs. We will reproduce the  by the below scenarios:

- While the client currently opens an NFS directory, the server deletes it.
- While the client currently opens an NFS directory, the server exports a different directory.

# Setup Lab
- Setup lab: [Network File System (NFS)](/html/lab/nfs.html).
- VM1 acts as NFS server.
- VM2 acts as NFS client.

:::::::::::::: {.columns}
::: {.column width=50%}

```sh
vm1$ exportfs -v
/srv/nfs 192.0.0.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```

:::
::: {.column width=50%}

```sh
vm2$ df -h
Filesystem                Size      Used Available Use% Mounted on
192.0.0.10:/srv/nfs       5.6G    221.0M      5.1G   4% /mnt/nfs
```

:::
::::::::::::::


# Delete an Open Directory

If the server deletes a file or directory while the client currently has open, the inode held by client becomes invalid.
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


# Re-Export an Open Directory
TODO...