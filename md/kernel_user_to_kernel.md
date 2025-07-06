---
title:  "From User Space to Kernel Space"
---


# 1. Kernel Module
Build a kernel module that:

- Registers a character device.
- Print log messages when device open and close.
- On read, sends the entire fixed message every time.
- Unregisters the device when the module is unloaded.

File: simplechar.c
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

File: Makefile.
```Makefile
  
obj-m += simplechar.o
EXTRA_CFLAGS += -g

all:
	make -C /root/kernel/linux-5.15.186 M=$(PWD) modules

clean:
	make -C /root/kernel/linux-5.15.186 M=$(PWD) clean
  
```

Build.
```sh
  
$ make
  
```


# 2. User Application
Build a user application that:

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

Build
```sh
  
$ gcc -static -o demo demo.c
  
```


# 2. Run
Copy the program into the filesystem that the QEMU VM boots from.
```sh
  
$ mkdir /mnt/rootfs
$ mount /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 /mnt/rootfs/

$ cp simplechar.ko demo /mnt/rootfs/opt/
  
```

Restart QEMU VM.
```sh
  
$ qemu-system-x86_64 -boot order=c -m 2048M \
    -kernel /root/kernel/linux-5.15.186/arch/x86/boot/bzImage \
    -hda /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 \
    -append "root=/dev/sda rw console=ttyS0,115200 nokaslr" \
    -nographic
  
```

On QEMU VM, insert module.
```sh
  
$ insmod /opt/simplechar.ko
[   28.060330] simplechar: loading out-of-tree module taints kernel.
[   28.075624] simplechar: registered major 248
  
```

Create device node on QEMU VM.
```sh
  
$ mknod /dev/simplechar c 248 0
$ chmod 666 /dev/simplechar
  
```

Run user application.
```sh
  
$ /opt/demo
[  232.892346] simplechar: opened
Read from kernel module: Hello from kernel module!
[  232.895639] simplechar: closed
  
```


# 2. Debug with GDB
Restart QEMU VM.
```sh
  
$ qemu-system-x86_64 -boot order=c -m 2048M \
    -kernel /root/kernel/linux-5.15.186/arch/x86/boot/bzImage \
    -hda /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 \
    -append "root=/dev/sda rw console=ttyS0,115200 nokaslr" \
    -nographic \
    -s -S
  
```

Run GDB.
```sh
  
$ gdb /root/kernel/linux-5.15.186/vmlinux

(gdb) target remote :1234
(gdb) continue
  
```

Insert module and setup device.
```sh
  
$ insmod /opt/simplechar.ko
[  111.235440] simplechar: loading out-of-tree module taints kernel.
[  111.250356] simplechar: registered major 248

$ mknod /dev/simplechar c 248 0
$ chmod 666 /dev/simplechar
  
```


In GDB, load debug symbols for kernel modules.
```sh
  
(gdb) lx-symbols
loading vmlinux
scanning for modules in /root/kernel/module
loading @0xffffffffc0000000: /root/kernel/module/simplechar.ko
  
```

In GDB, set breakpoint.
```sh
  
(gdb) break dev_read
Breakpoint 1 at 0xffffffffc0000000: file /root/kernel/module/simplechar.c, line 17.
  
```

On the QEMU VM, run the user application.
```sh
  
$ /opt/demo
  
```

Verify if the breakpoint is hit in GDB.
```sh
  
Breakpoint 1, dev_read (file=0xffff8880049d5800, buf=0x7ffeb62dcda0 "\340WL", len=127, 
    offset=0xffffc900001d7f08) at /root/kernel/module/simplechar.c:17
17          if (len < msg_len)
  
```