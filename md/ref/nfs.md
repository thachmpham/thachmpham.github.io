---
title: Network File System
---

## Lab

### Server

:::::::::::::: {.columns}
::: {.column width=50%}

Install nfs server.
```sh
$ apt install nfs-kernel-server
```

Start nfs server.
```sh
$ systemctl start nfs-kernel-server.service
```

Edit /etc/exports
```sh
/scratch *(rw,async,no_subtree_check,no_root_squash)
```

Create shared folder.
```sh
$ mkdir /scratch
```

Export.
```sh
$ exportfs -a
```

Show export table.
```sh
$ exportfs -v
```

:::
::: {.column width=50%}


:::
::::::::::::::

### Client

:::::::::::::: {.columns}
::: {.column width=50%}

Install nfs.

```sh
apt install nfs-common
```

Mount.
```sh
mkdir /mnt/scratch
mount localhost:/scratch /mnt/scratch
```

:::
::: {.column width=50%}


:::
::::::::::::::

## Useful Commands

:::::::::::::: {.columns}
::: {.column width=50%}

```sh
$ exportfs -v
$ exportfs -ua
$ showmount -e localhost
```

:::
::: {.column width=50%}


:::
::::::::::::::


## Reference
- [exportfs manual](https://linux.die.net/man/8/exportfs)
- [exports manual](https://linux.die.net/man/5/exports)