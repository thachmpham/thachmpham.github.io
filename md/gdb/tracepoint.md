---
title: "GDB: Tracepoints"
---

## Set Tracepoints
Observe the program without interrupting it.
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

## Collect Traces
Start trace data collection.
```sh
    tstart
```

Stop trace data collection.
```sh
    tstop
```

Show status.
```sh
    tstatus
```