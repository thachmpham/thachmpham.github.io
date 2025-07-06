---
title:  "Debug Linux Kernel with GDB"
---


# 1. Enable GDB Support
Enable GDB support with the config menu.
```sh
  
$ make menuconfig
  
```

- Kernel hacking
    - Kernel debugging
- Kernel hacking
    - Compile-time checks and compiler options
        - Compile the kernel with debug info
        - Provide GDB scripts for kernel debugging

```sh
  
$ make -j `nproc`
  
```


# 2. Run Kernel
Run the kernel with QEMU. 
```sh
  
$ qemu-system-x86_64 -boot order=c -m 2048M \
    -kernel /root/kernel/linux-5.15.186/arch/x86/boot/bzImage \
    -hda /root/kernel/buildroot-2025.05/output/images/rootfs.ext2 \
    -append "root=/dev/sda rw console=ttyS0,115200 nokaslr" \
    -nographic \
    -s -S
  
```

The -s -S options start QEMU with a GDB server on port 1234.


# 3. Attach GDB
To debug the Linux kernel remotely, launch GDB on the host and connect it to the QEMU GDB server.

```sh
  
$ gdb /root/kernel/linux-5.15.186/vmlinux

(gdb) target remote :1234
(gdb) continue
  
```


# 4. Breakpoint
Set a breakpoint at the `do_exit` function
```sh
  
(gdb) break do_exit
Breakpoint 60 at 0xffffffff810710a0: file kernel/exit.c, line 777.

(gdb) continue
  
```

In the QEMU VM, run the ls command.
```sh
  
$ ls
  
```

In GDB, verify that the breakpoint at do_exit is hit.
```sh
  
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