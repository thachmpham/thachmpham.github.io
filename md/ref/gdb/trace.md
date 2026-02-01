---
title: "GDB Tracepoint"
---

## Set Tracepoint
Observe the program without interrupting.
```sh
  trace [PROBE_MODIFIER] [LOCATION] [thread THREADNUM]
    [-force-condition] [if CONDITION]
```

- *force-condition*: defined even if condition invalid for current locations.


:::::::::::::: {.columns}
::: {.column width=50%}

Set actions to be taken at a trace.
```sh
  actions [tracepoint]
  > collect
  > while-stepping
  > end
  
```

:::
::: {.column width=50%}

Set data item to collect.
```sh
  collect expressions...
```

- *expressions*: variables, registers, $regs, $args, $locals, $_sdata.

:::
::::::::::::::


:::::::::::::: {.columns}
::: {.column width=50%}

Set stepping commands.
```sh
  > while-stepping N
    > collect ...
    > end
```

- *N*: step N instructions after the tracepoint, can collect data in each step.

:::
::: {.column width=50%}

Set passcount.

```sh
  passcount count [tracepoint]
```

- *count*: stop trace when passed 'count' times.

:::
::::::::::::::

<br>

Sample: Setup a tracepoint. Trace only work within remote mode, so need a gdbserver.

:::::::::::::: {.columns}
::: {.column width=40%}

```c
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

```sh

gdbserver --multi :12345
  
```

:::
::: {.column width=60%}

```sh
(gdb) target extended-remote :12345
(gdb) set remote exec-file demo
(gdb) file demo
(gdb) start


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

:::
::::::::::::::

<br>


## Collect Traces

:::::::::::::: {.columns}
::: {.column width=40%}

Start trace.
```sh
  tstart
```

Stop trace.
```sh
  tstop
```

Show status.
```sh
  tstatus
```

:::
::: {.column width=60%}

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
::::::::::::::

<br>


## Analyze Traces

:::::::::::::: {.columns}
::: {.column width=50%}

Select trace frame by index.
```sh
  tfind start # frame 0
  tfind n     # frame n
  tfind       # next frame
  tfind -     # prev frame
  
```

Select trace frame by filter.
```sh
  tfind tracepoint 
  tfind line         
  tfind pc         
  tfind range       
  tfind outside
  
```

Leave trace frame, back to live debug.
```sh
  tfind end
```

Print collected data.
```sh
  tdump
```

Convenience variables.
```sh
  (int) $trace_frame
  (int) $tracepoint
  (int) $trace_line
  (char []) $trace_file
  (char []) $trace_func
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

:::
::::::::::::::


## Save Trace Data

:::::::::::::: {.columns}
::: {.column width=50%}

Save trace data in client.
```sh
  tsave filename
```

Save trace data in server.
```sh
  tsave -r filename
```

- *-r*: remote.

Load trace data.
```sh
  target tfile filename
```

:::
::: {.column width=50%}

Sample: Save and load trace data.
```sh
(gdb) tsave demo.trace
```

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