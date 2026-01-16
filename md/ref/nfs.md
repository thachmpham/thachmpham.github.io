---
title: Network File System
---

## Lab

:::::::::::::: {.columns}
::: {.column width=50%}

### Server

```sh
$ apt install nfs-kernel-server
```

```sh
$ systemctl start nfs-kernel-server.service
```

Edit /etc/exports
```sh
/scratch *(rw,async,no_subtree_check,no_root_squash)
```

```sh
$ mkdir /scratch
```

```sh
$ exportfs -a
$ exportfs -v
```

:::
::: {.column width=50%}

### Client

```sh
apt install nfs-common
```

```sh
mkdir /mnt/scratch
mount localhost:/scratch /mnt/scratch
```

### Useful Commands

```sh
$ exportfs -v
$ exportfs -ua
$ showmount -e localhost
```

:::
::::::::::::::


## Reference
- [exportfs manual](https://linux.die.net/man/8/exportfs)
- [exports manual](https://linux.die.net/man/5/exports)