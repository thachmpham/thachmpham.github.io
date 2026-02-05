---
title: Linux Kernel Module
---

:::::::::::::: {.columns}
::: {.column}

| Module |         |
|:--------|:--------|
| `lsmod` | List module |
| `modinfo ip_tables` | Module info |
| `modinfo ip_tables.ko` | Module info |
| `insmod hello.ko` | Insert module |
| `rmmod hello`     | Remove module |

:::
::: {.column}

| printk  |         |
|:--------|:--------|
| `cat /proc/sys/kernel/printk` | Show log level |
| `echo 8 > /proc/sys/kernel/printk` | Set log level |
| `echo "4 4 1 7 " > /proc/sys/kernel/printk` | Set log level |
| `/etc/sysctl.conf, kernel.printk, sysctl -p` | Set permanent level |

:::
::: {.column}

| dmesg  |         |
|:--------|:--------|
| `dmesg` | Print kernel logs in buffer |
| `dmesg -l alert` | Print only alert logs |
| `dmesg -l 1` | Print only alert logs |
| `dmesg -l alert,err` | Print only alert, err logs |
| `dmesg -w` | Follow realtime logs |
| `dmesg -n alert` | Set log level |
| `dmesg -n 1` | Set log level |
| `dmesg -c` | Read and clear buffer |
| `dmesg -C` | Clear buffer |

:::
::::::::::::::