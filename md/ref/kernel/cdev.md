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
| IOCTL | API for users to query, configure a driver |

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

### Run

:::::::::::::: {.columns}
::: {.column}

```sh
$ git clone https://github.com/d0u9/Linux-Device-Driver.git
$ cd Linux-Device-Driver/eg_03_scull_basic
$ make
```

```sh
$ insmod scull.ko
```

```sh
# find major number.
$ cat /proc/devices
237 scull
```

```sh
# create device files.
$ mknod /dev/scull0 c 237 0
$ mknod /dev/scull1 c 237 1
$ mknod /dev/scull2 c 237 2
$ chmod 666 /dev/scull[0-2]
```

:::
::: {.column}

```sh
$ echo "hello" > /dev/scull0

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

:::
::: {.column}

Clean on exit.
```c
for(int i = 0; i < dev_count; i++)
{
	cdev_del(cdev);
}

devno = MKDEV(major_number, minor_number);
unregister_chrdev_region(from=devno, count=3);
```

Illustrate.
```go
┌─────────────┐      ┌─────────────┐
│   cdev      │      │   cdev      │
├─────────────┤      ├─────────────┤
│ /dev/scull0 │      │ /dev/scull1 │
│ major=237   │      │ major=237   │
│ minor=0     │      │ minor=1     │
└─────┬───────┘      └──────┬──────┘
      │cdev_init            │cdev_init
      │					 	│
      │  ┌───────────────┐  │  ┌────────────────┐
      └─►│file_operations│◄─┘  │ kernel module  │
         ├───────────────┤     │────────────────│
         │   .open       │     │ scull_open()   │
         │   .read       ├────►│ scull_read()   │
         │   .write      │     │ scull_write()  │
         │   .release    │     │ scull_release()│
         └───────────────┘     └────────────────┘
```

- When users open /dev/scull0, kernel will call scull_open().
- When users read /dev/scull0, kernel will call scull_read().
- The same with /dev/scull1.

:::
::::::::::::::

* * * * *

<br>


## Lab 2: IOCTL

### Run

:::::::::::::: {.columns}
::: {.column}

```sh
$ cd Linux-Device-Driver/eg_07_ioctl
$ make
```

```sh
$ insmod ioctl.ko
```

```sh
# find major number.
$ cat /proc/devices
237 ioctl
```

```sh
# create device files.
$ mknod /dev/ioctl c 237 0
$ chmod 666 /dev/ioctl
```

:::
::: {.column}

```sh
$ cd Linux-Device-Driver/eg_07_ioctl/test
$ make
```

```sh
$ ./ioctl_test.out
retval = 0

$ dmesg
ioctl_open() is invoked
ioctl_ioctl() is invoked
ioctl -> cmd: set print message
```

:::
::::::::::::::

### Code

:::::::::::::: {.columns}
::: {.column}

Define IOCTL commands.
```c
#include <ioctl.h>

#define IOCTL_DEV_NR	    1
#define IOCTL_IOC_MAGIC		'd'

#define IOCTL_RESET         _IO(IOCTL_IOC_MAGIC, 0)
#define IOCTL_HOWMANY       _IOWR(IOCTL_IOC_MAGIC, 1, int)
#define IOCTL_MESSAGE		_IOW(IOCTL_IOC_MAGIC, 2, int)
```

Setup file_operations.
```c
static struct file_operations fops = {
	.unlocked_ioctl = ioctl_ioctl,
};

long ioctl_ioctl(struct file *filp, unsigned int cmd,
            unsigned long arg)
{
    switch (cmd) {
    case IOCTL_RESET:
		pr_debug("ioctl -> cmd: reset\n");		
		break;
	case IOCTL_HOWMANY:
		pr_debug("ioctl -> cmd: set howmany\n");		
		break;
	case IOCTL_MESSAGE:
		pr_debug("ioctl -> cmd: set print message\n");
		break;	
	}
}
```

:::
::: {.column}

Test: Users call ioctl.
```c
int fd = open("/dev/ioctl", O_RDONLY);
ioctl(fd, IOCTL_HOWMANY, _);
ioctl(fd, IOCTL_MESSAGE, _);
ioctl(fd, IOCTL_RESET, _);
```

Illustrate.
```go
  ┌────────────┐                        
  │ cdev       │                        
  ├────────────┤                        
  │ /dev/ioctl │                        
  │ major=237  │                        
  │ minor=0    │                        
  └─────┬──────┘                        
        │cdev_init                      
        │                               
        ▼                               
┌─────────────────┐    ┌───────────────┐
│ file_operations │    │ kernel module │
│─────────────────│    │───────────────│
│ .unlocked_ioctl ├───►│ ioctl_ioctl() │
└─────────────────┘    └───────────────┘
```

- When users call ioctl on /dev/ioctl, kernel will call ioctl_ioctl().

:::
::::::::::::::

* * * * *

<br>

## Lab 3: MMAP

### Run
:::::::::::::: {.columns}
::: {.column}

```sh
$ Linux-Device-Driver/eg_23_simple
$ make
```

```sh
$ insmod simple.ko
```

```sh
# find major number
$ cat /proc/devices
237 simple
```

```sh
# create device files
$ mknod /dev/simple0 c 237 0
$ mknod /dev/simple1 c 237 1
$ chmod 666 /dev/simple*
```

:::
::: {.column}

```sh
$ Linux-Device-Driver/eg_23_simple/test
$ make
```

```sh
$ ./remap.out
$ dmesg
vm_start: 0x7fd01b9c5000, vm_end: 0x7fd01b9cb000
, len: 0x6000, vm_pgoff: 0x11e150(0x11e150000), ops=000000009da6972b
vma_open -- vm_start: 0x7fd01b9c5000, vm_end: 0x7fd01b9cb000
, len: 0x6000, vm_pgoff: 0x11e150(0x11e150000), ops=0000000023e1a403
pfn = 0, last page refc when close = 1
vma_close -- vm_start: 0x7fd01b9c5000, vm_end: 0x7fd01b9cb000
, len: 0x6000, vm_pgoff: 0x11e150(0x11e150000), ops=0000000023e1a403
```

```sh
$ ./fault.out
$ dmesg
vma_open -- vm_start: 0x7fc1b8c56000, vm_end: 0x7fc1b8c5c000
, len: 0x6000, vm_pgoff: 0x1(0x1000), ops=0000000035d509e0
pfn = 0, last page refc when close = 1
vma_close -- vm_start: 0x7fc1b8c56000, vm_end: 0x7fc1b8c5c000
, len: 0x6000, vm_pgoff: 0x1(0x1000), ops=0000000035d509e0
```

:::
::::::::::::::

### Code

:::::::::::::: {.columns}
::: {.column}

Setup file_operations.
```c
struct file_operations simple_remap_ops = {
	.open    = simple_open,
	.mmap    = simple_remap_mmap,
};

int simple_remap_mmap(struct file *filp,
			struct vm_area_struct *vma)
{
	pr_debug("vm_start:_",
		 vma->vm_start, vma->vm_end, len,
		 vma->vm_pgoff, vma->vm_pgoff << PAGE_SHIFT,
		 vma->vm_ops);

	vma->vm_ops = &simple_remap_vm_ops;
	simple_vma_open(vma);
}
```

Setup virtual memory operations.
```c
struct vm_operations_struct simple_remap_vm_ops = {
	.open =  simple_vma_open,
	.close = simple_vma_close,
};

void simple_vma_open(struct vm_area_struct *vma)
{
	size_t len = vma->vm_end - vma->vm_start;
	pr_debug("vma_open_",
		 vma->vm_start, vma->vm_end, len,
		 vma->vm_pgoff, vma->vm_pgoff << PAGE_SHIFT,
		 vma->vm_ops);
}

void simple_vma_close(struct vm_area_struct *vma)
{
	pr_debug("vma_close_",
		 vma->vm_start, vma->vm_end, len,
		 vma->vm_pgoff, vma->vm_pgoff << PAGE_SHIFT,
		 vma->vm_ops);
}
```

:::
::: {.column}

Test: Users call mmap.
```c
fd = open("/dev/simple0", _);
mmap(NULL, _, fd, _);
```

```go
     ┌──────────────┐                                  
     │     cdev     │                                  
     ├──────────────┤                                  
     │ /dev/simple0 │                                  
     │ major=237    │                                  
     │ minor=0      │                                  
     └─────┬────────┘                                  
           │cdev_init                                  
           │                                           
           ▼                                           
   ┌─────────────────┐                                 
   │ file_operations │                                 
   ├─────────────────│                                 
┌──┼──── .mmap       │                                 
│  └─────────────────┘                                 
│                             ┌────────────────────┐   
│                             │    kernel module   │   
│ ┌─────────────────────┐     │────────────────────│   
│ │    kernel module    │     │ simple_vma_open()  │   
│ │─────────────────────┤     │                    │◄─┐
└─► simple_remap_mmap() │     │ simple_vma_close() │  │
  └─────────┬───────────┘     └────────────────────┘  │
            │                                         │
            ▼                                         │
    ┌────────────────┐      ┌──────────────────────┐  │
    │ vm_area_struct │      │ vm_operations_struct │  │
    │────────────────│      │──────────────────────│  │
    │   .vm_ops ─────┼─────►│       .open          ┼──┘
    └────────────────┘      │       .close         │   
                            └──────────────────────┘   
```

- When users call mmap on /dev/simple0, kernel calls simple_remap_mmap().
- simple_remap_mmap() calls simple_vma_open() and create a vm_operations_struct.
- In vm_operations_struct:
	-  .open points to simple_vma_open().
	- .close points to simple_vma_close().
- When users call munmap on /dev/simple0, kernel calls simple_vma_close().

:::
::::::::::::::

## References
- [Linux Device Driver](https://github.com/d0u9/Linux-Device-Driver).