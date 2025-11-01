---
title: GDB Dump & Restore
---


GDB allows you to dump memory to a file and restore it later. This is useful for debugging, reverse engineering, or preserving the state of a program.

## 1. Program
```c++
  
#include <stdio.h>

char str[] = "hello world";

int main(int argc, char** argv)
{
    printf("%s\n", str);
    return 0;
}
  
```

```sh
  
$ g++ -g -o demo demo.cpp
$ gdb demo
  
```

```sh
  
(gdb) start

Temporary breakpoint 1, main (argc=1, argv=0x7fffffffdfa8) at demo.cpp:7
7           printf("%s\n", str);
  
```

## 2. Memory Map
Show memory map.
```sh
  
(gdb) info proc mappings 
process 4159
Mapped address spaces:

    Start Addr           End Addr       Size     Offset  Perms  objfile
0x555555554000     0x555555555000     0x1000        0x0  r--p   /root/demo/demo
0x555555555000     0x555555556000     0x1000     0x1000  r-xp   /root/demo/demo
0x555555556000     0x555555557000     0x1000     0x2000  r--p   /root/demo/demo
0x555555557000     0x555555558000     0x1000     0x2000  r--p   /root/demo/demo
0x555555558000     0x555555559000     0x1000     0x3000  rw-p   /root/demo/demo
0x7ffff7c00000     0x7ffff7c28000    0x28000        0x0  r--p   /usr/lib/x86_64-linux-gnu/libc.so.6
0x7ffff7c28000     0x7ffff7db0000   0x188000    0x28000  r-xp   /usr/lib/x86_64-linux-gnu/libc.so.6
0x7ffff7db0000     0x7ffff7dff000    0x4f000   0x1b0000  r--p   /usr/lib/x86_64-linux-gnu/libc.so.6
0x7ffff7dff000     0x7ffff7e03000     0x4000   0x1fe000  r--p   /usr/lib/x86_64-linux-gnu/libc.so.6
0x7ffff7e03000     0x7ffff7e05000     0x2000   0x202000  rw-p   /usr/lib/x86_64-linux-gnu/libc.so.6
0x7ffff7e05000     0x7ffff7e12000     0xd000        0x0  rw-p   
0x7ffff7fab000     0x7ffff7fae000     0x3000        0x0  rw-p   
0x7ffff7fbd000     0x7ffff7fbf000     0x2000        0x0  rw-p   
0x7ffff7fbf000     0x7ffff7fc3000     0x4000        0x0  r--p   [vvar]
0x7ffff7fc3000     0x7ffff7fc5000     0x2000        0x0  r-xp   [vdso]
0x7ffff7fc5000     0x7ffff7fc6000     0x1000        0x0  r--p   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
0x7ffff7fc6000     0x7ffff7ff1000    0x2b000     0x1000  r-xp   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
0x7ffff7ff1000     0x7ffff7ffb000     0xa000    0x2c000  r--p   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
0x7ffff7ffb000     0x7ffff7ffd000     0x2000    0x36000  r--p   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
0x7ffff7ffd000     0x7ffff7fff000     0x2000    0x38000  rw-p   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
0x7ffffffde000     0x7ffffffff000    0x21000        0x0  rw-p   [stack]
0xffffffffff600000 0xffffffffff601000     0x1000    0x0  --xp   [vsyscall]
  
```

Range of data segment.
```sh
  
0x555555558000     0x555555559000     0x1000     0x3000  rw-p   /root/demo/demo
  
```


## 3. Dump Memory
Dump data segment to file.
```sh
  
(gdb) dump memory mem.hex 0x555555558000 0x555555559000
  
```

```sh
  
$ hexdump -C mem.hex 
00000000  00 00 00 00 00 00 00 00  08 80 55 55 55 55 00 00  |..........UUUU..|
00000010  68 65 6c 6c 6f 20 77 6f  72 6c 64 00 00 00 00 00  |hello world.....|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
  
```

## 4. Restore Memory
```sh
  
(gdb) restore mem.hex binary 0x555555558000
Restoring binary file mem.hex into memory (0x555555558000 to 0x555555559000)
  
```


# References
- [GDB Copy Between Memory and a File](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Dump_002fRestore-Files.html)