---
title: 'GDB'
subtitle: '**Debug a Program Started by a Script**'
---


# 1. Introduction
Debugging a program launched by a script can be tricky. The script can have complex execution flow, and if the program runs in the background or in a separate shell, attaching a debugger to the detached process becomes more difficult.

Fortunately, the following GDB features can help:

- [Command line arguments `--args`](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Mode-Options.html#Mode-Options).
- [Catchpoints](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Set-Catchpoints.html).
- [Debugging Forks](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Forks.html)


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

- Break and debug when a process spawns a new program.
```sh
  
(gdb) catch exec
  
```

- Run.
```sh
  
(gdb) run
  
```

```sh
  
Starting program: /usr/bin/bash run.sh

====  begin   ====
[Attaching after Thread 0x7ffff7f73740 (LWP 42499) fork to child process 42502]
[New inferior 2 (process 42502)]

process 42502 is executing new program: /home/mm/demo/demo
[Switching to process 42502]

Thread 2.1 "demo" hit Catchpoint 1 (exec'd /home/mm/demo/demo), 0x00007ffff7fe4540 in ?? () from /lib64/ld-linux-x86-64.so.2
  
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
  
Continuing.

Thread 2.1 "demo" hit Breakpoint 2.1, main (argc=1, argv=0x7fffffffdc38) at demo.c:5
5           printf("hello\n");

# debugger entered the main() function
  
```

- Show inferiors.
```sh
  
(gdb) info inferiors
  
```

```sh
  
  Num  Description       Connection           Executable        
  1    process 42499     1 (native)           /usr/bin/bash     
* 2    process 42502     1 (native)           /home/mm/demo/demo
  
```


# References
- [sourceware.org/gdb](https://sourceware.org/gdb)
