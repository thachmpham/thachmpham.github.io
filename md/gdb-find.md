---
title: Search Memory With GDB Find
---

# 1. Introduction
The find command in GDB is used to search for specific byte sequences in a process's memory. It helps locate values, strings, or addresses within a given memory range.  
  
In Linux, when a program runs, it is loaded into memory and divided into different segments. Below are list of segments.  

| **Segment**   | **Description**                 | **Growth**  |
|:------------:|:--------------------------------:|:----------:|
| **Text** | Executable code, read-only, shared | Fixed      |
| **Data** | Initialized global/static variables | Fixed      |
| **BSS**  | Uninitialized global/static variables | Fixed      |
| **Heap**     | Dynamic memory | Upward     |
| **Stack**    | Local variables, function calls | Downward   |
| **Memory Map** | Shared libraries | Dynamic    |
| **Kernel**   | System calls, process management | Fixed      |


# 2. Lab
In this lab, we will search for memory in the segments of a process with gdb find command.

## 2.1. Program
Create file demo.c.
```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// data segment
char global_initialized[] = "hi data segment";

// bss segment
int global_uninitialized;

int main() {
    printf("hi text segment");

    // stack
    char local_variable[] = "hi stack";

    // heap
    char *heap_memory = (char *)malloc(50 * sizeof(char));    
    strcpy(heap_memory, "hi heap");    
    free(heap_memory);

    return 0;
}
  
```

Compile.
```sh
  
$ gcc -g -o demo demo.c
  
```

Start program with gdb.
```sh
  
$ gdb demo
(gdb) break 20
(gdb) run "hi command line argument"

Breakpoint 1, main () at demo.c:20
20          free(heap_memory);
  
```

Print proc mappings.
```sh
  
(gdb) info proc mappings

process 8711
Mapped address spaces:

    Start Addr           End Addr       Size     Offset  Perms  objfile
0x555555554000     0x555555555000     0x1000        0x0  r--p   /root/demo/demo
0x555555555000     0x555555556000     0x1000     0x1000  r-xp   /root/demo/demo
0x555555556000     0x555555557000     0x1000     0x2000  r--p   /root/demo/demo
0x555555557000     0x555555558000     0x1000     0x2000  r--p   /root/demo/demo
0x555555558000     0x555555559000     0x1000     0x3000  rw-p   /root/demo/demo
0x555555559000     0x55555557a000    0x21000        0x0  rw-p   [heap]
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

## 2.2. Text Segment
Range of text segment.
```sh
  
0x555555554000     0x555555555000     0x1000        0x0  r--p   /root/demo/demo
0x555555555000     0x555555556000     0x1000     0x1000  r-xp   /root/demo/demo
0x555555556000     0x555555557000     0x1000     0x2000  r--p   /root/demo/demo
0x555555557000     0x555555558000     0x1000     0x2000  r--p   /root/demo/demo
  
```

Find a string in text segment.
```sh
  
(gdb) find 0x555555554000, 0x555555558000-1, "hi text segment"
0x555555556004
0x555555557004
2 patterns found.

(gdb) print (char*) 0x555555556004
$10 = 0x555555556004 "hi text segment"

(gdb) print (char*) 0x555555557004
$11 = 0x555555557004 "hi text segment"
  
```

# Data Segment
Range of data segment.
```sh
  
0x555555558000     0x555555559000     0x1000     0x3000  rw-p   /root/demo/demo
  
```

Find a string in data segment.
```sh
  
(gdb) find 0x555555558000, 0x555555559000-1, "hi data segment"
0x555555558010 <global_initialized>
1 pattern found.

(gdb) print (char*) 0x555555558010
$12 = 0x555555558010 <global_initialized> "hi data segment"
  
```

# Stack
Range of stack segment.
```sh
  
0x7ffffffde000     0x7ffffffff000    0x21000        0x0  rw-p   [stack]
  
```

Find strings in stack segment.
```sh
  
(gdb) find 0x7ffffffde000, 0x7ffffffff000-1, "hi stack"
0x7fffffffde3f
1 pattern found.

(gdb) print (char*) 0x7fffffffde3f
$13 = 0x7fffffffde3f "hi stack"
  
```

```sh
  
(gdb) find 0x7ffffffde000, 0x7ffffffff000-1, "hi command line argument"
0x7fffffffe269
1 pattern found.

(gdb) print (char*) 0x7fffffffe269
$18 = 0x7fffffffe269 "hi command line argument"
  
```

# Heap
Range of heap segment.
``sh
  
0x555555559000     0x55555557a000    0x21000        0x0  rw-p   [heap]
  
```

Find a string in heap segment.
```sh
  
(gdb) find 0x555555559000, 0x55555557a000-1, "hi heap"
0x5555555596b0
1 pattern found.

(gdb) print (char*) 0x5555555596b0
$14 = 0x5555555596b0 "hi heap"
  
```


# References
- [GDB Searching Memory](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Searching-Memory.html)
- [Undo.io Search Memory With GDB](https://undo.io/resources/gdb-watchpoint/how-search-byte-sequence-memory-gdb-command-find)