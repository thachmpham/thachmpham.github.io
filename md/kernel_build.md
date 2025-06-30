---
title:  "Build & Run Linux Kernel"
---


# 1. Build Kernel
## 1.1. Docker Container
```Dockerfile
  
FROM ubuntu:22.04

RUN apt update
RUN apt install -y libncurses-dev make git exuberant-ctags bc libssl-dev
RUN apt install -y flex bison libelf-dev build-essential
RUN apt install -y cpio unzip rsync vim wget file xz-utils

CMD ["/bin/bash"]
  
```

```sh
  
$ docker build --tag kernel_builder .
  
```

```sh
  
$ mkdir /root/kernel
$ docker run --interactive --tty --volume /root/kernel:/root/kernel --name kbuilder kernel_builder
  
```

## 1.2. Build Kernel
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


# 2. Buildroot
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


# 3. Run Kernel
```sh
  
$ qemu-system-x86_64 -boot c -m 2048M \
    -kernel /root/kernel/linux-5.15.186/arch/x86/boot/bzImage \
    -hda /root/kernel/buildroot-2025.05/output/images/rootfs.ext4 \
    -append "root=/dev/sda rw console=ttyS0,115200 nokaslr" \
    -serial stdio -display none
  
```

- To exit QEMU, press "Ctrl + A", "X".
- To exit buildroot, press "Ctrl + C".
