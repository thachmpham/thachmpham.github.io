---
title: "GDB: Catchpoints"
---


## Set Catchpoints
Cause the debugger to stop for a event.
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

<br>

## Fork
:::::::::::::: {.columns}
::: {.column width=50%}

Stop on a call of fork or vfork.
```sh
catch fork
catch vfork

set detach-on-fork [on|off]
show detach-on-fork

set follow-fork-mode [parent|child]
show follow-fork-mode
```

Demo 
```sh
#include <unistd.h>
int main()
{    
  pid_t pid = fork();
  while (1) {}
}
```

:::
::: {.column width=50%}

```sh
(gdb) set detach-on-fork off
(gdb) catch fork

(gdb) run
Catchpoint 1 (forked process 4097), arch_fork (ctid=0x7ffff7d85a10)

(gdb) info inferiors 
  Num  Description       Connection           Executable        
* 1    process 4094      1 (native)           /root/demo/demo

(gdb) n
[New inferior 2 (process 4097)] arch_fork (ctid=0x7ffff7d85a10)

(gdb) info inferiors 
  Num  Description       Connection           Executable        
* 1    process 4094      1 (native)           /root/demo/demo   
  2    process 4097      1 (native)           /root/demo/demo
  
```

:::
::::::::::::::
