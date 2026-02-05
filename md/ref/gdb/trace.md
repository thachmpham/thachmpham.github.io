---
title: "GDB Tracepoint"
subtitle: "Observe program without interrupting"
---

:::::::::::::: {.columns}
::: {.column}

| Setup Trace |           |
|:----------- |:--------------------------|
| `trace demo.c:10` | Set tracepoint at demo.c line 10. |
| `actions`         | Set actions for current tracepoint. |
| `actions 3`       | Set actions for tracepoint 3. |
| `while-stepping 8`| Step 8 instructions, collect data each step |
| `collect exp`     | Data to collect when trace hit: variables, registers, etc. |
| `passcount 5 3`   | Stop after tracepoint 3 hit 5 times |
| `info tracepoints`   | Show tracepoint list |

<br>

| Collect Trace |           |
|:------------------|:--------------------------|
| `tstart` | Start trace collection |
| `tstop`  | Stop trace collection |
| `tstatus`| Status of collection |

<br>

| Manage Data |           |
|:------------------|:--------------------------|
| `tsave -r filename` | Save data to server |
| `tsave filename` | Save data to client |
| `target tfile filename` | Load trace data |

:::
::: {.column}

| Analyze data |           |
|:------------------|:--------------------------|
| `tfind start` | Frame 0 |
| `tfind n` | Frame n |
| `tfind` | Next frame |
| `tfind -` | Previous frame |
| `tfind end` | Back to live debug |
| `tfind tracepoint` | Filter by tracepoint |
| `tfind line` | Filter by line |
| `tfind pc` |  |
| `tfind range ` |  |
| `tfind outside` |  |
| `tdump` | Print collected data |

<br>

| Convenience variable |           |
|:------------------|:--------------------------|
| `(int) $trace_frame` | |
| `(int) $tracepoint` | |
| `(int) $trace_line` | |
| `(char []) $trace_file` | |
| `(char []) $trace_func` | |

:::
::::::::::::::

* * * * *

<br>

### Sample
:::::::::::::: {.columns}
::: {.column width=50%}

```c {.numberLines}
#include <unistd.h>

int main()
{
  int s = 0;
  int i = 0;
  while (1)
  {
    s += i;
    i += 1;
    sleep(3);
  }
}
```

Trace only work in remote mode, so start program under gdbserver.
```sh
$ gdbserver --multi :12345
```

```sh
(gdb) target extended-remote :12345
(gdb) set remote exec-file demo
(gdb) file demo
(gdb) start
```

Setup tracepoint.
```sh
(gdb) trace demo.c:10
(gdb) actions
> collect i, s
> while-stepping 2
  > collect i
  > end
> end
(gdb) passcount 20 2


(gdb) info tracepoints
```

Collect trace.
```sh
(gdb) tstart
(gdb) continue
(gdb) tstop

(gdb) tstatus
Trace stopped by a tstop.
Collected 24 trace frames.
```

:::
::: {.column width=50%}

Scan through the trace frames.
```sh
(gdb) tfind start
(gdb) while $trace_frame != -1
> frame
> tdump
> tfind
> end

Found trace frame 0, tracepoint 2
#0  0x00000000004006cc in main () at demo.c:10
Data collected at tracepoint 2, trace frame 0:
i = 16
s = 136

Found trace frame 1, tracepoint 2
#0  0x00000000004006d0 in main () at demo.c:10
Data collected at tracepoint 2, trace frame 1:
i = 16

Found trace frame 2, tracepoint 2
#0  0x00000000004006d4 in main () at demo.c:10
Data collected at tracepoint 2, trace frame 2:
i = 16

Found trace frame 3, tracepoint 2
#0  0x00000000004006cc in main () at demo.c:10
Data collected at tracepoint 2, trace frame 3:
i = 17
s = 153
```

Save collected data.
```sh
(gdb) tsave demo.trace
```

Load trace data.
```sh
(gdb) file demo
(gdb) target tfile demo.trace

(gdb) tfind start
Found trace frame 0, tracepoint 1
#0  0x00000000004006cc in main () at demo.c:10

(gdb) tdump
Data collected at tracepoint 1, trace frame 0:
i = 16
s = 136
```

:::
::::::::::::::

* * * * *

<br>