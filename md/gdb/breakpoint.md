---
title: "GDB: Breakpoints"
---

## Overview
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

## Software Breakpoints
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

## Manage Breakpoints

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