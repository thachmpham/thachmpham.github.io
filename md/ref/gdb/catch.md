---
title: "GDB Catchpoint"
subtitle: "Stop on a program event"
---


:::::::::::::: {.columns}
::: {.column}

| Fork |  |
|:-------------|:-------------|
| `catch fork`                | Stop execution on fork |
| `set detach-on-fork on`     | Detach child from gdb (default). |
| `set detach-on-fork off`    | Keep both parent, child under gdb. |
| `show detach-on-fork`       |           |
| `set follow-fork-mode parent` | Switch gdb to parent (default) |
| `set follow-fork-mode child`  | Switch gdb to child |
| `show follow-fork-mode`       |         |

:::
::: {.column}

| Exec |  |
|:-------------|:-------------|
| `catch exec`                | Stop execution on exec |
| `set follow-exec-mode same` | Keep gdb on the current process |
| `set follow-exec-mode new`  | Switch gdb to the new process |
| `show follow-exec-mode`     |   |

| Signal |  |
|:-------------|:-------------|
| `catch signal SIGSEGV`      | Stop execution on SIGSEGV |
| `catch signal all`          | Stop execution on all signals |

:::
::::::::::::::

* * * * *

<br>

### Tip 1: Keep Child Process Under Control.

:::::::::::::: {.columns}
::: {.column width=40%}

```sh
#include <unistd.h>
int main()
{
    fork();
    while (1) {}
}
```

```sh
(gdb) set detach-on-fork off
(gdb) catch fork
```

:::
::: {.column width=60%}

```sh
(gdb) run
Catchpoint 1 (forked process 4097), arch_fork (ctid=0x7ffff7d85a10)

(gdb) next
[New inferior 2 (process 4097)]

(gdb) info inferiors 
  Num  Description       Connection           Executable        
* 1    process 4094      1 (native)           /root/demo/demo   
  2    process 4097      1 (native)           /root/demo/demo
```

:::
::::::::::::::

* * * * *

<br>

### Tip 2: Debug Program Launched Script.

:::::::::::::: {.columns}
::: {.column width=40%}

```sh
# run.sh
# -----
ls /home
# -----
```

```sh
$ gdb --args bash run.sh

(gdb) set detach-on-fork off
(gdb) set follow-fork-mode child
(gdb) catch exec
```

:::
::: {.column width=60%}

```sh
(gdb) run
[New inferior 2 (process 5911)]
process 5911 is executing new program: /usr/bin/ls
[Switching to process 5911]
Thread 2.1 "ls" hit Catchpoint 1 (exec'd /usr/bin/ls')

(gdb) info inferiors 
  Num  Description       Connection           Executable        
  1    process 5909      1 (native)           /usr/bin/bash     
* 2    process 5911      1 (native)           /usr/bin/ls
```

:::
::::::::::::::

* * * * *

<br>

### Tip 3: Auto Print Backtrace On Crash.
:::::::::::::: {.columns}
::: {.column width=40%}

```c
void crash()
{
    int *p = 0;
    *p = 42;
}

int main()
{
    crash();
}
```

:::
::: {.column width=60%}

```sh
(gdb) catch signal SIGSEGV
(gdb) commands
>backtrace
>continue
>end

(gdb) run
Catchpoint 1 (signal SIGSEGV)
#0  0x000000000040067c in crash () at demo.c:4
#1  0x0000000000400698 in main () at demo.c:9
Program terminated with signal SIGSEGV, Segmentation fault.
```

:::
::::::::::::::

* * * * *

<br>