---
title:  "From User Space to Kernel Space"
---


# Kernel Module

:::::::::::::: {.columns}
::: {.column width=40%}

The kernel module does the below tasks:

- Registers a character device.
- Print log messages when device open and close.
- On read syscall, reply a fixed message to user space.
- Unregisters the device when the module is unloaded.


1. Makefile.

```Makefile
obj-m += simplechar.o
EXTRA_CFLAGS += -g

all:
	make -C /root/kernel/linux-5.15.186 M=$(PWD) modules

clean:
	make -C /root/kernel/linux-5.15.186 M=$(PWD) clean
```

2. Build.
```sh
$ make
```

3. Copy module to filesystem of VM.
```sh
$ mkdir /mnt/rootfs
$ mount /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 /mnt/rootfs/

$ cp simplechar.ko /mnt/rootfs/opt/
```

4. Start VM.
```sh
$ qemu-system-x86_64 -boot order=c -m 2048M \
    -kernel /root/kernel/linux-5.15.186/arch/x86/boot/bzImage \
    -hda /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 \
    -append "root=/dev/sda rw console=ttyS0,115200 nokaslr" \
    -nographic
```

5. On VM, insert the module.
```sh
$ insmod /opt/simplechar.ko
[   28.060330] simplechar: loading out-of-tree module taints kernel.
[   28.075624] simplechar: registered major 248
```

6. Create device on VM.
```sh
$ mknod /dev/simplechar c 248 0
$ chmod 666 /dev/simplechar
```

:::
::: {.column width=60%}

- File: simplechar.c
```c
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/uaccess.h>

#define DEVICE_NAME "simplechar"
#define MESSAGE "Hello from kernel module!\n"

static int major;
static int msg_len = sizeof(MESSAGE) - 1;

static int dev_open(struct inode *inode, struct file *file) {
    printk(KERN_INFO "simplechar: opened\n");
    return 0;
}

static ssize_t dev_read(struct file *file, char __user *buf, size_t len, loff_t *offset) {
    if (len < msg_len)
        return -EINVAL; // user buffer too small

    if (copy_to_user(buf, MESSAGE, msg_len))
        return -EFAULT;

    return msg_len;
}

static int dev_release(struct inode *inode, struct file *file) {
    printk(KERN_INFO "simplechar: closed\n");
    return 0;
}

static struct file_operations fops = {
    .open = dev_open,
    .read = dev_read,
    .release = dev_release,
};

static int __init simplechar_init(void) {
    major = register_chrdev(0, DEVICE_NAME, &fops);
    if (major < 0)
        return major;
    printk(KERN_INFO "simplechar: registered major %d\n", major);
    return 0;
}

static void __exit simplechar_exit(void) {
    unregister_chrdev(major, DEVICE_NAME);
    printk(KERN_INFO "simplechar: unregistered\n");
}

MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Simple char device module");

module_init(simplechar_init);
module_exit(simplechar_exit);
```

:::
::::::::::::::

# User-Space Program

:::::::::::::: {.columns}
::: {.column width=40%}

1. Build
```sh
$ gcc -static -o demo demo.c
```

2. Copy the program to the filesystem of VM.
```sh
$ cp demo /mnt/rootfs/opt/
```

3. Restart QEMU VM.
```sh
$ qemu-system-x86_64 -boot order=c -m 2048M \
    -kernel /root/kernel/linux-5.15.186/arch/x86/boot/bzImage \
    -hda /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 \
    -append "root=/dev/sda rw console=ttyS0,115200 nokaslr" \
    -nographic
```

4. Setup module, device.
```sh
$ insmod /opt/simplechar.ko
$ mknod /dev/simplechar c 248 0
$ chmod 666 /dev/simplechar
```

5. Run program.
```sh
$ /opt/demo
[  232.892346] simplechar: opened
Read from kernel module: Hello from kernel module!
[  232.895639] simplechar: closed
```

:::
::: {.column width=60%}

The user-space program does the below tasks:

- Opens /dev/simplechar for reading.
- Reads message from the device.
- Prints the message to the screen.
- Closes the device.

File: demo.c
```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>

#define DEVICE "/dev/simplechar"

int main() {
    int fd = open(DEVICE, O_RDONLY);
    if (fd < 0) {
        perror("Failed to open device");
        return 1;
    }

    char buf[128];
    ssize_t ret = read(fd, buf, sizeof(buf) - 1);
    if (ret < 0) {
        perror("Failed to read");
        close(fd);
        return 1;
    }

    buf[ret] = '\0';
    printf("Read from kernel module: %s", buf);

    close(fd);
    return 0;
} 
```

:::
::::::::::::::


# Debug with GDB

:::::::::::::: {.columns}
::: {.column width=50%}

1. Start QEMU VM with gdb server.
```sh
$ qemu-system-x86_64 -boot order=c -m 2048M \
    -kernel /root/kernel/linux-5.15.186/arch/x86/boot/bzImage \
    -hda /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 \
    -append "root=/dev/sda rw console=ttyS0,115200 nokaslr" \
    -nographic \
    -s -S
```

2. From host, remote debug to VM.
```sh
$ gdb /root/kernel/linux-5.15.186/vmlinux
(gdb) target remote :1234
```

- Load debug symbols of modules.
```sh
(gdb) lx-symbols
loading vmlinux
scanning for modules in /root/kernel/module
loading @0xffffffffc0000000: /root/kernel/module/simplechar.ko
```

- Set breakpoint.
```sh
(gdb) break dev_read
Breakpoint 1 at 0xffffffffc0000000: file /root/kernel/module/simplechar.c, line 17.
(gdb) continue
```

:::
::: {.column width=60%}

4. On VM, setup module, device.
```sh
$ insmod /opt/simplechar.ko
[  111.235440] simplechar: loading out-of-tree module taints kernel.
[  111.250356] simplechar: registered major 248

$ mknod /dev/simplechar c 248 0
$ chmod 666 /dev/simplechar
```

5. Run the program.
```sh
$ /opt/demo
```

- The breakpoint hits.
```sh
Breakpoint 1, dev_read (file=0xffff8880049d5800, buf=0x7ffeb62dcda0 "\340WL", len=127, 
    offset=0xffffc900001d7f08) at /root/kernel/module/simplechar.c:17
17          if (len < msg_len)
```

:::
::::::::::::::