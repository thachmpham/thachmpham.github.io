---
title:  "Linux Kernel: Build & Run"
---

# 1. Build Kernel

:::::::::::::: {.columns}
::: {.column}

Set up a docker container with essential packages.

1. Dockerfile.

```Dockerfile
FROM ubuntu:22.04

RUN apt update
RUN apt install -y libncurses-dev make git exuberant-ctags bc libssl-dev
RUN apt install -y flex bison libelf-dev build-essential
RUN apt install -y cpio unzip rsync vim wget file xz-utils

CMD ["/bin/bash"]
```

2. Build docker image.
```sh
$ docker build --tag kernel_builder .
```

3. Run docker container.
```sh
$ mkdir /root/kernel
$ docker run -it -v /root/kernel:/root/kernel --name kbuilder kernel_builder
```

:::
::: {.column}

4. Download kernel source code.
```sh
$ cd /root/kernel
$ wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.15.186.tar.xz
$ tar xf linux-5.15.186.tar.xz
```

5. Build kernel.
```sh
$ docker exec -it kbuilder bash

$ cd /root/kernel/linux-5.15.186
$ make defconfig
$ make -j `nproc`
```

Output.
```sh
BUILD   arch/x86/boot/bzImage
Kernel: arch/x86/boot/bzImage is ready  (#1)
```

6. Run kernel with QEMU.
```sh
$ qemu-system-x86_64 -boot order=c -m 2048M \
    -kernel /root/kernel/linux-5.15.186/arch/x86/boot/bzImage \    
    -append "root=/dev/sda rw console=ttyS0,115200 nokaslr" \
    -nographic
```

:::
::::::::::::::

# 2. Build Filesystem Image

:::::::::::::: {.columns}
::: {.column}

1. Download buildroot.
```sh
$ cd /root/kernel
$ wget https://buildroot.org/downloads/buildroot-2025.05.tar.gz
$ tar xf buildroot-2025.05.tar.gz
```

2. Build buildroot in the same docker container as kernel.
```sh
$ cd buildroot-2025.05
$ make defconfig
```

```sh
$ make menuconfig
```

In the menu config, select below options:

- Target options -> Target Architecture -> x86_64
- Toolchain -> C library -> glibc
- Filesystem images -> ext4 root filesystem
- Target Packages -> Hardware Handling -> pciutils
- System Configuration -> Run a getty (login prompt) after boot -> TTY port -> ttyS0

:::
::: {.column}

3. Build.

```sh
$ export FORCE_UNSAFE_CONFIGURE=1
$ make -j `nproc`
```

4. Run the kernel on QEMU, with the filesystem image.
```sh
$ qemu-system-x86_64 -boot order=c -m 2048M \
    -kernel /root/kernel/linux-5.15.186/arch/x86/boot/bzImage \
    -hda /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 \
    -append "root=/dev/sda rw console=ttyS0,115200 nokaslr" \
    -nographic
```

To exit QEMU: Ctrl + A, X.

:::
::::::::::::::

# 3. Hello World Program

:::::::::::::: {.columns}
::: {.column width=50%}

1. Build the program on the host, then copy it to the QEMU VM and run it there.

- File: hello.c
```c
#include <stdio.h>
int main() {
    printf("Hello, Linux kernel on QEMU!\n");
    return 0;
}
```

- The Linux kernel does not provide the C standard library (libc.so.6) or the dynamic linker (ld-linux.so), as they belong to user space. To run program in a minimal kernel setup without these components, static linking is used to bundle all dependencies into a single self-contained binary.

```sh
$ gcc -static -o hello hello.c
```

2. Copy the program from host into the filesystem of the QEMU VM.
```sh
$ mkdir /mnt/rootfs
$ mount /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 /mnt/rootfs/

$ cp hello /mnt/rootfs/opt
```

:::
::: {.column width=50%}

3. Restart QEMU VM.
```sh
$ qemu-system-x86_64 -boot order=c -m 2048M \
    -kernel /root/kernel/linux-5.15.186/arch/x86/boot/bzImage \
    -hda /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 \
    -append "root=/dev/sda rw console=ttyS0,115200 nokaslr" \
    -nographic
```

4. Run the program on QEMU VM.
```sh
$ /opt/demo
Hello, Linux kernel on QEMU!
```

:::
::::::::::::::