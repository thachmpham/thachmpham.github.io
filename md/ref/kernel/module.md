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

| API |         |
|:--------|:--------|
| `module_init` | Specify entry function |
| `module_exit` | Specify exit function |
| `module_param` | Module parameter |

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
::: {.column}

| printk configuration |         |
|:--------|:--------|
| `cat /proc/sys/kernel/printk` | Show log level |
| `echo 8 > /proc/sys/kernel/printk` | Set log level |
| `echo "4 4 1 7 " > /proc/sys/kernel/printk` | Set log level |
| `/etc/sysctl.conf, kernel.printk, sysctl -p` | Set permanent level |

:::
::::::::::::::

* * * * *

<br>

### Lab 1: Hello World

:::::::::::::: {.columns}
::: {.column}

Environment.
```sh
$ apt install linux-headers-`uname -r`
$ apt install build-essential kmod
```

hello.c
```c
#include <linux/module.h>

static int __init hello_init(void)
{
	printk("hello_init\n");
	return 0;
}

static void __exit hello_exit(void)
{
	printk("hello_exit\n");
}

module_init(hello_init);
module_exit(hello_exit);

MODULE_LICENSE("GPL");
```

:::
::: {.column}

Makefile
```sh
obj-m += hello.o

KDIR := /lib/modules/$(shell uname -r)/build
PWD  := $(shell pwd)

all:
	$(MAKE) -C $(KDIR) M=$(PWD) modules

clean:
	$(MAKE) -C $(KDIR) M=$(PWD) clean
```

```sh
$ make V=1
```

Play.
```sh
$ modinfo hello.ko
$ insmod hello.ko
$ lsmod
$ rmmod hello
$ dmesg
```

:::
::::::::::::::

* * * * *

<br>

### Lab 2: Module Parameters

:::::::::::::: {.columns}
::: {.column}

hello.c
```c
#include <linux/module.h>

static char *name = "alex";
static int age = 20;

module_param(name, charp, S_IRUGO);
module_param(age, int, S_IRUGO);

static int __init hello_init(void)
{
	printk("hello_init name=%s, age=%d\n", name, age);
	return 0;
}

static void __exit hello_exit(void)
{
	printk("hello_exit\n");
}

module_init(hello_init);
module_exit(hello_exit);

MODULE_LICENSE("GPL");
```

:::
::: {.column}

Makefile
```sh
obj-m += hello.o

KDIR := /lib/modules/$(shell uname -r)/build
PWD  := $(shell pwd)

all:
	$(MAKE) -C $(KDIR) M=$(PWD) modules

clean:
	$(MAKE) -C $(KDIR) M=$(PWD) clean
```

```sh
$ make V=1
```

Play.
```sh
$ insmod hello.ko
$ rmmod hello

$ insmod hello.ko name=bob age=30 
$ dmesg
```

:::
::::::::::::::

* * * * *

<br>

## References
- [printk](https://docs.kernel.org/core-api/printk-basics.html).
- [Linux Device Driver](https://github.com/d0u9/Linux-Device-Driver).
- [Linux Kernel Module Programming Guide](https://sysprog21.github.io/lkmpg).
