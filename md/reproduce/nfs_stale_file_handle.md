---
title: 'NFS Stale File Handle'
subtitle: '(Reproduce Series)'
---


# Overview
When an NFS client accesses an inode that no longer available on the server, the stale file handle error (ESTALE) occurs. 

Reproduce cases:

- While the client currently opens an NFS directory, the server deletes it.
- While the client currently opens an NFS directory, the server exports a different directory.

# Prepare Lab
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
- Scenario: While the client open a directory, the server deletes the directory.
- Result:
    - The inode held by the client becomes unavailable.
    - The further request from the client to the inode will fail with 'Stale file handle' error.

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
vm2$ ls
ls: .: Stale file handle
```

:::
::::::::::::::


# Export a Different Directory
- Scenario: While the client open a directory, the server unexport the directory and export a different path.
- Result:
    - The underlying inodes on server change.
    - The further request from the client to the inode will fail with 'Stale file handle' error.


:::::::::::::: {.columns}
::: {.column width=50%}

**Server**. 

- Step 1: Exporting the /srv/nfs directory.
```sh
vm1$ exportfs -v
/srv/nfs        192.0.0.0/24(sync,wdelay,hide,no_subtree_check,fsid=1,sec=sys,rw,secure,root_squash,no_all_squash)
```

- Step 3: Export a different directory.
```sh
vm1$ mkdir /srv/nfs1
vm1$ vi /etc/exports
# ---------------------------------------------------------------------- #
# /srv/nfs 192.0.0.0/24(rw,sync,no_subtree_check,fsid=1)    # old directory
/srv/nfs1 192.0.0.0/24(rw,sync,no_subtree_check,fsid=1)     # new directory
# ---------------------------------------------------------------------- #

vm1$ exportfs -va
exporting 192.0.0.0/24:/srv/nfs1
```

:::
::: {.column width=50%}

**Client**. 

- Step 2: Go to the directory.
```sh
vm2$ cd /mnt/nfs
```

- Step 4: List files.
```sh
vm2$ ls
ls: .: Stale file handle
```

:::
::::::::::::::
