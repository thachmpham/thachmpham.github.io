---
title:  "Build & Run Linux Kernel"
---

Contents:

- Build kernel with docker.
- Build a filesystem image with buildroot.
- Run kernel with qemu.


# 1. Docker
Set up a docker container as a build machine with essential packages installed.

```Dockerfile
  
FROM ubuntu:22.04

RUN apt update
RUN apt install -y libncurses-dev make git exuberant-ctags bc libssl-dev
RUN apt install -y flex bison libelf-dev build-essential
RUN apt install -y cpio unzip rsync vim wget file xz-utils

CMD ["/bin/bash"]
  
```


# 2. Build Kernel
Build the Linux kernel using the docker container.

```sh
  
$ docker build --tag kernel_builder .
  
```

```sh
  
$ mkdir /root/kernel
$ docker run -it -v /root/kernel:/root/kernel --name kbuilder kernel_builder
  
```

```sh
  
$ cd /root/kernel
$ wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.15.186.tar.xz
$ tar xf linux-5.15.186.tar.xz
  
```

```sh
  
$ docker exec --interactive --tty kbuilder bash

$ cd /root/kernel/linux-5.15.186
$ make defconfig
$ make -j `nproc`
  
```

- Output.
```sh
  
BUILD   arch/x86/boot/bzImage
Kernel: arch/x86/boot/bzImage is ready  (#1)
  
```


# 3. Build Filesystem Image
Build an EXT filesystem image using **buildroot** and the docker container.
```sh
  
$ cd /root/kernel
$ wget https://buildroot.org/downloads/buildroot-2025.05.tar.gz
$ tar xf buildroot-2025.05.tar.gz
  
```

```sh
  
$ cd buildroot-2025.05
$ make defconfig
  
```

```sh
  
$ make menuconfig
  
```

- In the menu config, select below options:
    - Target options -> Target Architecture -> x86_64
    - Toolchain -> C library -> glibc
    - Filesystem images -> ext4 root filesystem
    - Target Packages -> Hardware Handling -> pciutils
    - System Configuration -> Run a getty (login prompt) after boot -> TTY port -> ttyS0

```sh
  
$ export FORCE_UNSAFE_CONFIGURE=1
$ make -j `nproc`
  
```


# 4. Run
Run the kernel with QEMU.
```sh
  
$ qemu-system-x86_64 -boot order=c -m 2048M \
    -kernel /root/kernel/linux-5.15.186/arch/x86/boot/bzImage \
    -hda /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 \
    -append "root=/dev/sda rw console=ttyS0,115200 nokaslr" \
    -nographic
  
```

- To exit QEMU: Ctrl + A, X.


# 5. Hello World Program
Build the program on the host, then copy it to the QEMU VM and run it there.

File: hello.c
```c
  
#include <stdio.h>

int main() {
    printf("Hello, Linux kernel on QEMU!\n");
    return 0;
}
  
```

The Linux kernel does not provide the C standard library (libc.so.6) or the dynamic linker (ld-linux.so), as they belong to user space.
To run program in a minimal kernel setup without these components, static linking is used to bundle all dependencies into a single self-contained binary.

```sh
  
$ gcc -static -o hello hello.c
  
```

On the host, copy the program into the filesystem that the QEMU VM boots from.
```sh
  
$ mkdir /mnt/rootfs
$ mount /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 /mnt/rootfs/

$ cp hello /mnt/rootfs/opt
  
```

Restart QEMU VM.
```sh
  
$ qemu-system-x86_64 -boot order=c -m 2048M \
    -kernel /root/kernel/linux-5.15.186/arch/x86/boot/bzImage \
    -hda /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 \
    -append "root=/dev/sda rw console=ttyS0,115200 nokaslr" \
    -nographic
  
```

Run the program on QEMU VM.
```sh
  
$ /opt/demo
Hello, Linux kernel on QEMU!
  
```