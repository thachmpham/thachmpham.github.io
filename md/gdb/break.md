---
title: "GBD Breakpoints"
subtitle: "Stop at a location"
---


# Cheatsheet

:::::::::::::: {.columns}
::: {.column}

| Linespec Location |           |
|:------------------|-----------|
| `break 10`        | Line 10 current file |
| `break hello.c::10`| Line 10, file hello.c |
| `break -3`         | Current line -3 |
| `break +3`         | Current line +3 |
| `break sum`        | Function sum    |
| `break hello.c::sum` | Function sum in hello.c |
| `break A::sum`       | Function A::sum |


| Explicit Location       |             |
|:------------------------|:------------|
| `break -qualified sum `    | Match sum, not A::sum |


| Address Location        |             |
|:------------------------|:------------|
|`break *sum+16`          | Instruction 16 in sum |
|`break 0x`0x400670       | Instruction at 0x400670 |


| Regex Location          |             |
|:------------------------|:------------|
|`rbreak sum.*`           | Function sum.* |
|`rbreak demo.c:sum.*`    | Function sum.* in hello.c       |
|`rbreak .`               | All function in current program |
|`rbreak demo.c:.`        | All function in file            |


| Condition               |             |
|:------------------------|:------------|
| `break sum if x > 3`    | Conditional       |
| `break sum inferior 1`  | Inferior specific |
| `break sum inferior 1 if x > 3`  | Inferior + condition |
| `break sum thread 1`    | Thread specific   |
| `break sum thread 1 if x > 3`    | Thread + condition |
| `condition 2 if x > 5`    | Change condition of break 2 |

:::
::: {.column}

| Command                 |             |
|:------------------------|:------------|
| `command`               | Command for current break |
| `command 2`             | Command for break 2 |

```sh
command 2
    info args
    backtrace
    continue
end
```

| Temporary Break   |           |
|:------------------|-----------|
| `tbreak sum`      | Auto delete after hit |


| Management|           |
|:----------|:----------|
| `info break`          | List of breakpoints |
| `enable break [ids]`  | Enable breakpoints  |
| `disable break [ids]` | Disable breakpoints |
| `delete break [ids]`  | Delete breakpoints  |
| `clear <locspec>`     | Delete all breakpoints at locspec |
| `save break <file>`   | Save breakpoints to file      |
| `source <file>`       | Load breakpoints from file    |


| Tips & Tricks |  |
|:-------------|:-------------|
| Break before process exits  | `break _exit` |

:::
::::::::::::::

:::::::::::::: {.columns}
::: {.column width=50%}

Software Breakpoint.

- To setup a software breakpoint, GDB replaces the program instruction with a trap, illegal divide or some other instructions that will cause an exception. When the exception raised, GDB catches it and stops the program.
- The number of software breakpoints is unlimited.
- To set a software breakpoint, the target address must reside in a writable memory region.

:::
::: {.column width=50%}

Hardware Breakpoint.

- To setup a hardware breakpoint, GDB stores the breakpoint address to a debug register. When the program counter $pc matches the value in the debug register, GDB stops the program.
- The number of debug registers are limited and varies depending on the architecture. For example, x86 has 4 debug registers, DR0-DR3. So, the number of hardward breakpoints is limited.
- A hardware breakpoint can be placed at an address in read-only memory region.

:::
::::::::::::::

<br>

# Tips & Tricks
## Debug Code in MMAP Regions

:::::::::::::: {.columns}
::: {.column width=40%}

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

Illustrate math.o and process memory.
```go
+---------------------+
|       ........      |
+---------------------+
| .text               | 0x0040
|  + sum()            |         + 0x0000
|  + sub()            |         + 0x0018
+---------------------+
```

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


# References
- [GDB Internal, Breakpoint Handling](https://sourceware.org/gdb/wiki/Internals/Breakpoint%20Handling)