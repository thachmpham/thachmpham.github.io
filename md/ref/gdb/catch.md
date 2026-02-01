---
title: "GDB Catchpoints"
---

## Usages

* * * * *

### Catchpoints
Stop on program events, such as C++ exceptions or the loading of a shared library, etc.
```sh
catch assert    # Catch failed Ada assertions, when raised.
catch catch     # Catch an exception, when caught.
catch exception # Catch Ada exceptions, when raised.
catch exec      # Catch calls to exec.
catch fork      # Catch calls to fork.
catch handlers  # Catch Ada exceptions, when handled.
catch load      # Catch loads of shared libraries.
catch rethrow   # Catch an exception, when rethrown.
catch signal    # Catch signals by their names and/or numbers.
catch syscall   # Catch system calls by their names, groups and/or numbers.
catch throw     # Catch an exception, when thrown.
catch unload    # Catch unloads of shared libraries.
catch vfork     # Catch calls to vfork
```

Show catchpoints.
```sh
show breakpoint
```

<br>

### Fork

:::::::::::::: {.columns}
::: {.column}

```sh
catch fork
catch vfork

set detach-on-fork [on|off]
set follow-fork-mode [parent|child]

show detach-on-fork
show follow-fork-mode
```

:::
::: {.column}

```sh
#include <unistd.h>

int main()
{    
  fork();
  while (1) {}
}
```

:::
::: {.column}

```sh
(gdb) set detach-on-fork off
(gdb) catch fork
(gdb) run
Catchpoint 1 (forked process 4097), arch_fork (ctid=0x7ffff7d85a10)
(gdb) n
[New inferior 2 (process 4097)]
(gdb) info inferiors 
  Num  Description       Connection           Executable        
* 1    process 4094      1 (native)           /root/demo/demo   
  2    process 4097      1 (native)           /root/demo/demo
```

:::
::::::::::::::

### Exec

:::::::::::::: {.columns}
::: {.column}

```sh
catch exec
set follow-exec-mode [new|same]
show follow-exec-mode
```

:::
::: {.column}

```sh
# run.sh
# -----
ls /home
# -----
```

:::
::: {.column}

```sh
$ gdb --args bash run.sh
(gdb) set detach-on-fork off
(gdb) set follow-fork-mode child
(gdb) catch exec
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

<br>

### Syscall
```sh
catch syscall [name | number | group:groupname | g:groupname] …
ausyscall --dump
```

<br>

### Signal
:::::::::::::: {.columns}
::: {.column width=30%}

One reason that catch signal can be more useful than handle is that you can attach commands and conditions to the catchpoint.
```sh
catch signal [signal… | ‘all’]
kill -l
```

:::
::: {.column width=30%}

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

Automatically print backtrace before program terminated.

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

### Library Load
Stop on the loading or unloading of a shared library.
```sh
  catch load [regexp]
  catch unload [regexp]
```