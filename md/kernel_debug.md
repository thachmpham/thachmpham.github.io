---
title:  "Debug Linux Kernel"
---


# 1. GDB
## 1.1. Enable GDB Support
```sh
  
$ make menuconfig
  
```

- Enable GDB Support.
```sh
  
Kernel hacking  ->  Kernel debugging

Kernel hacking  ->  Compile-time checks and compiler options
                    ->  Compile the kernel with debug info
                    ->  Provide GDB scripts for kernel debugging
  
```

```sh
  
$ make -j `nproc`
  
```

## 1.2. Run Kernel
```sh
  
$ qemu-system-x86_64 -boot c -m 2048M \
    -kernel /root/kernel/linux-5.15.186/arch/x86/boot/bzImage \
    -hda /root/kernel/buildroot-2025.05/output/images/rootfs.ext4 \
    -append "root=/dev/sda rw console=ttyS0,115200 nokaslr" \
    -serial stdio -display none -s -S
  
```

## 1.3. Run GDB
```sh
  
$ gdb /root/kernel/linux-5.15.186/vmlinux

(gdb) target remote :1234
(gdb) continue
  
```

## 1.4. Break Point
```sh
  
(gdb) break do_exit
Breakpoint 60 at 0xffffffff810710a0: file kernel/exit.c, line 777.
  
```

```sh
  
$ ls
  
```

```sh
  
(gdb) continue
Continuing.

Breakpoint 60, do_exit (code=code@entry=0) at kernel/exit.c:777
777     {
(gdb) bt
#0  do_exit (code=code@entry=0) at kernel/exit.c:777
#1  0xffffffff81072c0e in do_group_exit (exit_code=0) at kernel/exit.c:997
#2  0xffffffff81072c8f in __do_sys_exit_group (error_code=<optimized out>) at kernel/exit.c:1008
#3  __se_sys_exit_group (error_code=<optimized out>) at kernel/exit.c:1006
#4  __ia32_sys_exit_group (regs=<optimized out>) at kernel/exit.c:1006
#5  0xffffffff81c333d5 in do_syscall_32_irqs_on (nr=<optimized out>, regs=0xffffc900001d7f58) at arch/x86/entry/common.c:112
#6  __do_fast_syscall_32 (regs=regs@entry=0xffffc900001d7f58) at arch/x86/entry/common.c:178
#7  0xffffffff81c3358f in do_fast_syscall_32 (regs=0xffffc900001d7f58) at arch/x86/entry/common.c:203
#8  0xffffffff81e01749 in entry_SYSCALL_compat () at arch/x86/entry/entry_64_compat.S:272
#9  0x0000000000000000 in ?? ()
  
```