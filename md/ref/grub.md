---
title: "GRUB"
---

:::::::::::::: {.columns}
::: {.column width=50%}

Introduction.

GRUB is a bootloader responsible for loading vmlinuz and initrd.img into RAM at booting:

- vmlinuz is the linux kernel.
- initrd.img contains tools and drivers to mount root filesystem.

:::
::: {.column width=50%}

Configuration files.

- `/etc/default/grub`
    - User settings.
- `/etc/grub.d`
    - Helper files.
- `/boot/grub2/grub.cfg`
    - Read by grub at boot time.
    - Generated from /etc/default/grub.

:::
::::::::::::::


Parameters.

| Parameters | Meaning | Examples |
|:-----------|:-------|:--------|
| GRUB_TIMEOUT          | Timeout for grub menu | -1, 0, 5                  |
| GRUB_TIMEOUT_STYLE    | Menu style            | menu, countdown, hidden   |
| GRUB_DEFAULT          | Default entry to boot | Index or ID of meny entry |
| GRUB_CMDLINE_LINUX    | Command-line arguments to vmlinuz | console=ttyS0 |
| GRUB_DISABLE_OS_PROBER| Generate menu entries for other OS | true, false |


:::::::::::::: {.columns}
::: {.column width=50%}

Update configuration.
```sh
# edit config
$ vi /etc/default/grub

# validate config
$ grub-script-check -v /etc/default/grub

# apply change
$ grub-mkconfig -o /boot/grub/grub.cfg
```

Install grub to a device.

```sh
# install grub
$ grub-install /dev/sdb

# verify
$ xxd -l 512 /dev/sdb
```

:::
::: {.column width=50%}

Master Boot Record.

- Master Boot Record (MBR) is the first 512 bytes of a disk.
- grub-install writes boot code to MBR.
- When power on, BIOS executes the boot code to start the boot process.

| Offset | Size         | Purpose                                  |
|:-------|:-------------|:-----------------------------------------|
| 0x0000  | 446 B        | Boot code                                |
| 0x01be  | 64  B        | Partition table (4 entries × 16B)        |
| 0x01fe  | 2   B        | Boot signature (0x55aa)                  |

:::
::::::::::::::


### Reference
- [GRUB Manual](https://www.gnu.org/software/grub/manual/grub/grub.html).
- [Linux Kernel Parameters](https://www.kernel.org/doc/html/v4.14/admin-guide/kernel-parameters.html).