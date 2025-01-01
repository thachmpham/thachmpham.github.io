---
title: 'GDB'
subtitle: '**Debug A Program Started By A Script**'
---


# 1. Introduction
When debugging a program, things can get tricky if it is launched via a script. The script might set environment variables, pass specific arguments, or even invoke the program indirectly, making it challenging to attach GDB.


# 2. Lab
## 2.1. Program
- File demo.c.
```c
  
#include <stdio.h>

int main(int argc, char** argv)
{
    printf("hello\n");
    return 0;
}
  
```

- Build.
```sh
  
$ gcc -g demo.c -o demo
  
```

- Script run.sh.
```sh
  
echo "====  begin   ===="
./demo
echo "==== end     ===="
  
```


## 2.2. Debug
- Launch script with GDB.
```sh
  
$ gdb --args bash run.sh
  
```

- Prevent automatically detaching from the child process when the script forks, allow to debug both the parent and child processes.
```sh
  
(gdb) set detach-on-fork off
  
```

- Follow and continue debugging the child process after a fork, rather than the parent.
```sh
  
(gdb) set follow-fork-mode child
  
```

- Catch the exec event to pause execution when a new process is spawned.
```sh
  
(gdb) catch exec
  
```

- Run.
```sh
  
(gdb) run
  
```

```sh
  
# console log
Starting program: /usr/bin/bash run.sh

====  begin   ====
[Attaching after Thread 0x7ffff7f73740 (LWP 42499) fork to child process 42502]
[New inferior 2 (process 42502)]
process 42502 is executing new program: /opt/app/demo
[Switching to process 42502]

Thread 2.1 "demo" hit Catchpoint 1 (exec'd /opt/app/demo), 0x00007ffff7fe4540 in ?? () from /lib64/ld-linux-x86-64.so.2
  
```

- Set breakpoint.
```sh
  
(gdb) break main
  
```

- Continue.
```sh
  
(gdb) continue
  
```

```sh
  
# console log
Continuing.

Thread 2.1 "demo" hit Breakpoint 2.1, main (argc=1, argv=0x7fffffffdc38) at demo.c:5
5           printf("hello\n");
  
```

- Show inferiors.
```sh
  
(gdb) info inferiors
  
```

```sh
  
# console log
  Num  Description       Connection           Executable        
  1    process 42499     1 (native)           /usr/bin/bash     
* 2    process 42502     1 (native)           /opt/app/demo
  
```

- Continue the demo program.
```sh
  
(gdb) continue
  
```

```sh
  
# console log
Continuing.
hello
[Inferior 2 (process 42502) exited normally]
  
```

- The demo program already exited.
```sh
  
(gdb) info inferiors
  
```

```sh
  
# console log
  Num  Description       Connection           Executable        
  1    process 42499     1 (native)           /usr/bin/bash     
* 2    <null>  42502     1 (native)           /opt/app/demo
  
```

- Move debugger to the script.
```sh
  
(gdb) inferior 1
  
```

```sh
  
# console log
[Switching to inferior 1 [process 42499] (/usr/bin/bash)]
[Switching to thread 1.1 (Thread 0x7ffff7f73740 (LWP 42499))]
  
```

- Continue the script.
```sh
  
(gdb) continue
  
```

```sh
  
# console log
Continuing.
==== end     ====
[Inferior 1 (process 42499) exited normally]
  
```


# 3. Cheatsheets
- GDB script.
```sh
  
set confirm off
set pagination off

set detach-on-fork off
set follow-fork-mode child

catch exec
  
```

- GDB.
```sh
  
$ gdb -q -x debug.gdb --args bash run.sh
  
```


# References
- [GDB Command Line Arguments](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Mode-Options.html#Mode-Options).
- [GDB Catchpoints](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Set-Catchpoints.html).
- [GDB Debugging Forks](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Forks.html)