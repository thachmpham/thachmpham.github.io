---
title: "Character Device Driver"
---

:::::::::::::: {.columns}
::: {.column}

| Function				|     								     |
|:----------------------|:---------------------------------------|
| cdev_init			    | Init char device |
| cdev_add 			    | Add char device to system |
| cdev_del 			    | Remove char device from system |
| alloc_chrdev_region   | Register a range of char device numbers |
| unregister_chrdev_region | Undo alloc_chrdev_region |

| Command |  		 |
|:--------|----------|
| mknod   | Make device files |

:::
::: {.column}

| Struct |  		|
|:-------|----------|
| inode  | Identity of file on disk |
| file   | Open instance of file |
| file_operations | Tells kernel how to handle IO requests |

| Term	  |  		 |
|:--------|----------|
| Major number | Kernel driver ID, who responsible for devices |
| Minor number | ID of devices within a driver |

:::
::: {.column}

<pre class="mermaid">
erDiagram
	Driver ||--o{ Device : handle

	Driver {
		int majorNumber
	}

	Device {
		int minorNumber
	}
</pre>

:::
::::::::::::::

* * * * *

<br>

## Lab 1: Scull Basic

:::::::::::::: {.columns}
::: {.column}

### Build
```sh
$ git clone https://github.com/d0u9/Linux-Device-Driver.git
$ cd Linux-Device-Driver/eg_03_scull_basic
$ make
```


### Run

```sh
$ insmod scull.ko
```

Find major number.
```sh
$ cat /proc/devices
237 scull
```

Create device files.
```sh
$ mknod /dev/scull0 c 237 0
$ mknod /dev/scull1 c 237 1
$ mknod /dev/scull2 c 237 2
```

Set permissions.
```sh
$ chmod 666 /dev/scull[0-2]
```

:::
::: {.column}

Test.
```sh
$ echo "hello" > /dev/scull0
hello

$ dmesg
scull_open() is invoked
scull_trim() is invoked
scull_write() is invoked
WR pos = 6, block = 0, offset = 0, write 6 bytes
scull_release() is invoked
```

```sh
$ cat /dev/scull0
hello

$ dmesg
scull_open() is invoked
scull_read() is invoked
RD pos = 6, block = 0, offset = 0, read 6 bytes
scull_read() is invoked
RD pos = 6, block = 0, offset = 6, read 131072 bytes
scull_release() is invoked
```

:::
::::::::::::::

### Code

:::::::::::::: {.columns}
::: {.column}

Setup file_operations.
```c
struct file_operations scull_fops = {
	.owner   = _,
	.open    = scull_open,
	.read    = scull_read,
	.write   = scull_write,
	.release = scull_release,
};

int scull_open(struct inode *inode, struct file *filp) {}
int scull_release(struct inode *inode, struct file *filp) {}
ssize_t scull_read(struct file *filp, char __user *buff, _) {}
ssize_t scull_write(struct file *filp, const char __user *buff, _ ) {}
```

:::
::: {.column}

Setup char devices.
```c
// register number of devices that driver can handle
alloc_chrdev_region(_, _, dev_count=3, name="scull");

// create devices with associated file_operations, 
// major_number, minor_number
for(int i = 0; i < dev_count; i++)
{
	// init a char device
	cdev_init(cdev, &scull_fops);
	
	// make device id from major, minor numbers
	devno = MKDEV(major_number, minor_number);

	// add device to system
	cdev_add(cdev, devno, _);
}
```

Clean on exit.
```c
for(int i = 0; i < dev_count; i++)
{
	cdev_del(cdev);
}

devno = MKDEV(major_number, minor_number);
unregister_chrdev_region(from=devno, count=3);
```

:::
::::::::::::::

* * * * *

<br>