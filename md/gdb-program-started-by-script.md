---
title: 'GDB'
subtitle: '**Debug A Program Started By A Script**'
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


## 2.2. Debug with GDB
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
* 2    process 42502     1 (native)           /opt/app/demo
  
```


## 2.3. GDB & GDBServer
During debugging, the program's output to the GDB terminal can trigger a SIGTTOU signal, causing GDB to stop.

To prevent this, we can use gdb with gdbserver, allows GDB to run in one terminal while the program executes in a separate terminal, ensuring the program's output does not interfere with the debugging session.

### 2.3.1. GDBServer on Terminal 1
- Launch script with gdbserver.
```sh
  
term1$ gdbserver localhost:5555 bash run.sh
  
```

```sh
  
Process bash created; pid = 53908
Listening on port 5555
  
```

### 2.3.2. GDB on Terminal 2
- Attach gdb.
```sh
  
term2$ gdb -ex "target remote localhost:5555"
  
```

```sh
  
Remote debugging using localhost:5555
Reading /opt/app/demo from remote target...
  
```

- Configure gdb.
```sh
  
(gdb) set detach-on-fork off
(gdb) set follow-fork-mode child
(gdb) catch exec
  
```

- Continue.
```sh
  
(gdb) continue
  
```

```sh
  
Continuing.
[Attaching after Thread 53908.53908 fork to child Thread 54343.54343]
[New inferior 2 (process 54343)]
process 54343 is executing new program: /opt/app/demo
[Switching to Thread 54343.54343]

Thread 2.1 hit Catchpoint 1 (exec'd /opt/app/demo), 0x00007ffff7fe4540 in ?? ()
   from target:/lib64/ld-linux-x86-64.so.2
  
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

Thread 2.1 hit Breakpoint 2.1, main (argc=1, argv=0x7fffffffdc58) at demo.c:5
5           printf("hello\n");
  
```

- Show inferiors.
```sh
  
(gdb) info inferiors
  
```

```sh
  
Thread 2.1 hit Breakpoint 2.1, main (argc=1, argv=0x7fffffffdc58) at demo.c:5
5           printf("hello\n");
(gdb) info inferiors
  Num  Description       Connection                Executable        
  1    process 53908     1 (remote localhost:5555) target:/usr/bin/bash 
* 2    process 54343     1 (remote localhost:5555) target:/opt/app/demo
  
```


# References
- [sourceware.org/gdb](https://sourceware.org/gdb)
