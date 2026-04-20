---
title: "GBD Breakpoints"
subtitle: "Stop at a location"
---

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

**References**  

- [GDB Internal, Breakpoint Handling](https://sourceware.org/gdb/wiki/Internals/Breakpoint%20Handling)