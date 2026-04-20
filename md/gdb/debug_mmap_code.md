---
title: 'Debug Code in mmap Memory'
---


:::::::::::::: {.columns}
::: {.column width=50%}

1. math.c
```c {.numberLines}
int sum(int x, int y)
{
    return x + y;
}

int sub(int x, int y)
{
    return x - y;
}
```

2. main.c
```c {.numberLines}
#include <sys/mman.h>
#include <stddef.h>
#include <fcntl.h>
#include <stdio.h>

int main()
{
    // load math.o to a new memory region
    int fd = open("math.o", O_RDONLY);
    void *addr = mmap(
        NULL, // kernel choose start addr for region
        4096, // length of the region
        PROT_READ|PROT_EXEC,    // protection mode
        MAP_PRIVATE,            // visible mode
        fd,                     // file to load
        0                       // offset in file
    );

    // find address of .text segment
    void *text = addr + 0x0040;

    // find address of sum and sub functions
    void *sum = text + 0x0000;
    void *sub = text + 0x0018;

    // call sum(5, 4)
    int isum = ((int (*)(int, int))sum)(5, 4);

    // call sub(5, 4)
    int isum = ((int (*)(int, int))sub)(5, 4);

    printf("isum=%d, isub=%d\n", isum, isum);
    // output: isum=9, isum=1

    return 0;
}
```

3. Build and run.
```sh
$ gcc -c math.c -o math.o
$ gcc -g main.c -o main
$ ./main
```

:::
::: {.column width=50%}

4. Inspect math.o
```sh
$ readelf --section-headers --wide math.o
  [Nr] Name     Type        Address             Off     Size    ES  Flg Lk Inf Al
  [ 1] .text    PROGBITS    0000000000000000    000040  00002e  00  AX  0   0  1

$ readelf --symbols math.o
   Num:    Value          Size Type    Bind   Vis      Ndx Name     
     3: 0000000000000000    24 FUNC    GLOBAL DEFAULT    1 sum
     4: 0000000000000018    22 FUNC    GLOBAL DEFAULT    1 sub
```

5. Inspect process memory.
```sh
(gdb) break main.c:27
(gdb) run
(gdb) info proc mappings
      Start Addr           End Addr       Size     Offset  Perms  objfile
  0x7ffff7ffa000     0x7ffff7ffb000     0x1000        0x0  r-xp   math.o
```

Illustrate math.o.
```go
+---------------------+
|       ........      |
+---------------------+
| .text               | 0x0040
|  + sum()            |         + 0x0000
|  + sub()            |         + 0x0018
+---------------------+
```

Illustrate process memory.
```go
+---------------------+
|       ........      |
+---------------------+
| math.o              | 0xfffff7ff5000
|   .text             | 0xfffff7ff5040
|   ├── sum()         |         + 0x0000
|   └── sub()         |         + 0x0018
+---------------------+
```

6. Add symbols and hbreak.
```sh
(gdb) add-symbol-file math.o 0x7ffff7ffa040
(gdb) info function sub
0x00007ffff7ffa058  sub

(gdb) hbreak sub
Hardware assisted breakpoint 5 at 0x7ffff7ffa060

(gdb) continue
Breakpoint 5, 0x00007ffff7ffa060 in sub ()

(gdb) backtrace
#0  0x00007ffff7ffa060 in sub ()
#1  0x000055555555521d in main () at main.c:30
```

:::
::::::::::::::
