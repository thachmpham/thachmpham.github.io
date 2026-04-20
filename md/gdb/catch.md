---
title: "GDB Catchpoint"
subtitle: "Stop on a program event"
---

# Cheatsheet

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

| Exec |  |
|:-------------|:-------------|
| `catch exec`                | Stop execution on exec |
| `set follow-exec-mode same` | Keep gdb on the current process |
| `set follow-exec-mode new`  | Switch gdb to the new process |
| `show follow-exec-mode`     |   |

:::
::: {.column}

| Signal |  |
|:-------------|:-------------|
| `catch signal SIGSEGV`      | Stop execution on SIGSEGV |
| `catch signal all`          | Stop execution on all signals |

| Tips & Tricks |  |
|:-------------|:-------------|
| Catch before process exits  | `catch syscall exit` |
|                             | `catch syscall exit_group` |

:::
::::::::::::::


# Tips & Tricks
## Fork

1. Keep Child Process Under GDB Control.

:::::::::::::: {.columns}
::: {.column width=50%}


:::
::::::::::::::


2. Debug Programs Started By Scripts.

:::::::::::::: {.columns}
::: {.column width=50%}

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
::: {.column width=50%}

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


## Signal

1. Automatically Print Backtrace On Crash.

:::::::::::::: {.columns}
::: {.column width=50%}

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
::: {.column width=50%}

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

