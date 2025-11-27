---
title: "GDB: Breakpoints"
---

## 1. Overview
GDB supports two types of breakpoints: software and hardware breakpoints.

:::::::::::::: {.columns}
::: {.column width=50%}

**Software Breakpoint.**

<span style="color: yellow">Mechanism:</span>
To setup a software breakpoint, GDB replaces the program instruction with a trap, illegal divide or some other instructions that will cause an exception. When the exception raised, GDB catches it and stops the program.

<span style="color: yellow">Maximum:</span>
The number of software breakpoints is unlimited.

<br><br><br>

<span style="color: yellow">Permission:</span>
To set a software breakpoint, the target address must reside in a writable memory region.

<br><br><br>

<span style="color: yellow">Usage:</span>
Typically used for regular debugging, where performance and memory permissions are not critical.

:::
::: {.column width=50%}

**Hardware Breakpoint.**

<span style="color: yellow">Mechanism:</span>
To setup a hardware breakpoint, GDB stores the breakpoint address to a debug register. When the program counter $pc matches the value in the debug register, GDB stops the program.

<span style="color: yellow">Maximum:</span>
The number of debug registers are limited and varies depending on the architecture. For example, x86 has 4 debug registers, DR0-DR3. So, the number of hardward breakpoints is limited.

<span style="color: yellow">Permission:</span>
A hardware breakpoint can be placed at an address in read-only memory region.

<span style="color: yellow">Performance:</span>
The impact of a hardware breakpoint on program performance is smaller than that of a software breakpoint.

<span style="color: yellow">Usage:</span>
Typically used for read-only code or in live system, where performance and memory safety are critical.

:::
::::::::::::::

<br>

## 2. Software Breakpoints
<span style="color: yellow">Breakpoint:</span>
Stop program whenever a point in the program is reached.

```sh
  break [-qualified] locspec
```
- *locspec*: line number, function name, address.
- *qualified*: full match.

:::::::::::::: {.columns}
::: {.column width=50%}

```c {.numberLines}
int sum(int a, int b)
{
  return a + b;
}

int main()
{
  int a = 1, b = 2;
  int s = sum(a, b);
}
```

:::
::: {.column width=50%}

```sh
(gdb) break 3
(gdb) break sum
(gdb) break demo.c:3
(gdb) break demo.c:sum

(gdb) break -qualified exit

(gdb) disassemble sum
   0x000000000040066c <+0>:     sub     sp, sp, #0x10
   0x0000000000400670 <+4>:     str     w0, [sp, #12]
   0x0000000000400674 <+8>:     str     w1, [sp, #8]
   0x0000000000400678 <+12>:    ldr     w1, [sp, #12]
   0x000000000040067c <+16>:    ldr     w0, [sp, #8]
   0x0000000000400680 <+20>:    add     w0, w1, w0
   0x0000000000400684 <+24>:    add     sp, sp, #0x10
   0x0000000000400688 <+28>:    ret

(gdb) break *sum+4
(gdb) break 0x0000000000400670
  
```

:::
::::::::::::::

<span style="color: yellow">Regex Breakpoints:</span>
Break by regular expression.

:::::::::::::: {.columns}
::: {.column width=50%}

Set breakpoints on all functions matching the regular expression regex
```sh
  rbeak regex
  rbreak file:regex
```

- *regex*: use standard regular expression, like grep.

:::
::: {.column width=50%}

```sh
rbreak sum.*
rbreak demo.c:sum.*

# all functions in current program
rbreak .
# all functions in file
rbreak demo.c:.
```

:::
::::::::::::::

<br>

<span style="color: yellow">Conditional Breakpoints:</span>
Only stop on certain conditions.

Set a conditional breakpoint.
```sh
  break locspec [-force-condition] if cond
```

- *cond*: only stop at breakpoint when cond is true.
- *force-condition*: force to enable to cond.

Change condition.
```sh
  condition breakpoint [cond]
```

<br>

<span style="color: yellow">Inferior-Specific Breakpoints:</span>
Only stop on certain processes.

```sh
  break locspec inferior inferior-id [if cond]
```

<br>

<span style="color: yellow">Thread-Specific Breakpoints:</span>
Only stop on certain threads.

```sh
  break locspec thread thread-id [if cond]
```

<br>

<span style="color: yellow">Temporary Breakpoints:</span>
Automatically deleted after the hit.
```sh
  tbreak args
```

<br>

<span style="color: yellow">Commands:</span>
Automate debugging actions when a breakpoint is hit.

:::::::::::::: {.columns}
::: {.column width=50%}

Set commands.
```sh
  commands [breakpoints...]
      command
      ...
  end
```

:::
::: {.column width=50%}

Change commands.
```sh
  commands breakpoints...
      command
      ...
  end
```

:::
::::::::::::::

<br>

## 3. Hardware Breakpoints
Set a hardware assisted breakpoint.
```sh
  hbreak [LOCATION] [thread THREADNUM]
         [-force-condition] [if CONDITION]
```

Set a temporary hardware assisted breakpoint.
```sh
  thbreak [LOCATION] [thread THREADNUM]
          [-force-condition] [if CONDITION]
```

<br>

<span style="color: yellow">Sample:</span>
Set hbreak at an instruction in a read-only mmap region.

:::::::::::::: {.columns}
::: {.column width=50%}

File math.c
```c
int sum(int x, int y)
{
    return x + y;
}

int sub(int x, int y)
{
    return x - y;
}
```

:::
::: {.column width=50%}

Build.
```sh
$ gcc -c math.c -o math.o
```

:::
::::::::::::::

Find the offset of the .text section. The output shows it located at 0x000040.
```sh
$ readelf --section-headers --wide math.o
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  [ 1] .text             PROGBITS        0000000000000000 000040 00002e 00  AX  0   0  1
  
```

Find the offsets of the sum and sub functions. The output shows sum at 0x0000 and sub at 0x0020, these are relative offsets to the start of the .text section.
```sh
$ readelf --symbols math.o
Symbol table '.symtab' contains 5 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name     
     3: 0000000000000000    24 FUNC    GLOBAL DEFAULT    1 sum
     4: 0000000000000018    22 FUNC    GLOBAL DEFAULT    1 sub
  
```

Illustration of the math.o structure.
```go
+---------------------+
| ELF Header          | At: 0x0000
+---------------------+
| Program Header      |
+---------------------+
| .text section       | At: 0x0040
|  + sum()            | At: 0x0040 + 0x0000
|  + sub()            | At: 0x0040 + 0x0018
+---------------------+
|       ........      | Other sections
+---------------------+
| Section Header      |
+---------------------+
  
```

File main.c
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
        NULL,  // staring address of memory region, NULL: kernel will choose
        4096,   // length of the region
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
    int a = ((int (*)(int, int))sum)(5, 4);

    // call sub(5, 4)
    int b = ((int (*)(int, int))sub)(5, 4);

    printf("a=%d, b=%d\n", a, b);
    // output: a=9, b=1

    return 0;
}
  
```

Build.
```sh
$ gcc -g main.c -o main
  
```

Set hbreak.
```sh

(gdb) file main
(gdb) break main.c:27
(gdb) run

(gdb) info proc mappings
      Start Addr           End Addr       Size     Offset  Perms  objfile
  0x7ffff7ffa000     0x7ffff7ffb000     0x1000        0x0  r-xp   math.o

# add-symbol-file <file> <.text address in memory>
# .text addr = region addr + .text offset in ELF
#              = 0x7ffff7ffa000 + 0x0040
#              = 0x7ffff7ffa040
(gdb) add-symbol-file math.o 0x7ffff7ffa040
add symbol table from file "math.o" at
        .text_addr = 0x7ffff7ffa040
Reading symbols from math.o...

(gdb) info function sub
0x00007ffff7ffa058  sub

(gdb) info symbol sub
sub in section .text of math.o

(gdb) hbreak sub
Hardware assisted breakpoint 5 at 0x7ffff7ffa060

(gdb) continue
Breakpoint 5, 0x00007ffff7ffa060 in sub ()

(gdb) bt
#0  0x00007ffff7ffa060 in sub ()
#1  0x000055555555521d in main () at main.c:30

(gdb) continue
Continuing.
a=9, b=1
  
```

Illustration of the memory mapping in the process.
```go
+---------------------+
| /root/demo/main     | At: 0x0000000000400000
| + main()            | At: 0x000000000040072c
+---------------------+
| ........            | (Other regions)
|                     |
+---------------------+
| /root/demo/math.o   | At: 0x0000fffff7ff5000 (region addr)
|   .text             | At: 0x0000fffff7ff5040 (region addr + .text offset)
|  + sum()            | At: 0x0000fffff7ff5040 (.text addr + sum offset)
|  + sub()            | At: 0x0000fffff7ff5058 (.text addr + sub offset)
+---------------------+
| ........            | (Other regions)
|                     |
+---------------------+
  
```

<br>

## 4. Management

:::::::::::::: {.columns}
::: {.column width=50%}

Show breakpoints
```sh
  info break
```

Enable, disable breakpoints
```sh
  enable breakpoints [breakpoints]
  disable breakpoints [breakpoints]
```

Save, restore breakpoints.
```sh
  save breakpoints file
  source file
```

:::
::: {.column width=50%}

Delete breakpoints.
```sh
  delete [breakpoints] [list…]
```

Delete all breakpoints.
```sh
  delete breakpoints
```

Delete all breakpoints at to locspec.
```sh
  clear locspec
```

:::
::::::::::::::

## Reference
- [GDB Internal, Breakpoint Handling](https://sourceware.org/gdb/wiki/Internals/Breakpoint%20Handling)