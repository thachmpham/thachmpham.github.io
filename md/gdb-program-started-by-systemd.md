---
title: 'GDB'
subtitle: '**Debug A Program Started By A Systemd Service**'
---


# 1. Introduction
Debugging a program launched by a systemd service can be challenging due to its execution flow, the program runs in the background, attaching a debugger to the process started by systemd becomes trickier.

In this post, we will examine how to debug with gdbserver and gdb.


# 2. Lab
## 2.1. Systemd Service
- File /opt/demo/main.c.
```c
  
#include <stdlib.h>
#include <unistd.h>
#include <syslog.h>

int main(int argc, char** argv)
{
        openlog("main", LOG_PID | LOG_CONS, LOG_USER);
        syslog(LOG_INFO, "started");

        int n = 0;
        while (1)
        {
                syslog(LOG_INFO, "running... %d", n);
                n += 1;
                sleep(3);
        }

        closelog();
        return 0;
}
  
```

- Build.
```sh
  
$ gcc -g main.c -o main
  
```

- File /opt/demo/control.sh.
```sh
  
#!/bin/bash

start() {
    start-stop-daemon --start --background \
        --make-pidfile --pidfile /var/run/demo.pid \
        --exec /opt/demo/main
}

stop() {
    start-stop-daemon --stop --pidfile /var/run/demo.pid
}

case $1 in
    start)
        start;;
    stop)
        stop;;
    *)
        echo 'Usage: control.sh {start|stop}'
        exit 1;;
esac

exit 0
  
```

- File /etc/systemd/system/demo.service.
```sh
  
[Unit]
Description=Demo Service

[Service]
ExecStart=/opt/demo/control.sh start
ExecStop=/opt/demo/control.sh stop
Type=forking
KillMode=non
    
```

We will debug the program in two phases:

- Debug the entry point of the program when the service is started (Section 2.2).
- Debug the program while it is running (Section 2.3).


## 2.2. Debug Entrypoint
- Modify start() and stop() functions in control.sh script.
```sh
  
start() {
    start-stop-daemon --start --background \
        --make-pidfile --pidfile /var/run/demo.pid \
        --exec /usr/bin/gdbserver localhost:5555 /opt/demo/main
}

stop() {
    kill -9 $(cat /var/run/demo.pid)
}
  
```

- Start service.
```sh
  
$ systemctl start demo
  
```

```sh
  
# syslog
systemd[1]: Starting demo.service - Demo Service...
systemd[1]: Started demo.service - Demo Service.
  
```

- Attach gdb.
```sh
  
$ gdb -q -ex "target extended-remote localhost:5555"
  
```

- Show processes.
```sh
  
info inferiors
  
```

```sh
  
  Num  Description       Connection                         Executable        
* 1    process 86099     1 (extended-remote localhost:5555) target:/opt/demo/main
  
```

- Set breakpoints.
```sh
  
(gdb) break main
  
```

```sh
  
Breakpoint 1 at 0x55555555519c: file main.c, line 7.
  
```

- Continue.
```sh
  
(gdb) continue
  
```

```sh
  
Continuing.

Breakpoint 1, main (argc=1, argv=0x7fffffffec78) at main.c:7
7               openlog("main", LOG_PID | LOG_CONS, LOG_USER);
  
```

- Continue
```sh
  
(gdb) continue
  
```

```sh
  
# syslog
main[86099]: started
main[86099]: running... 0
main[86099]: running... 1
main[86099]: running... 2
  
```

- Detach.
```sh
  
(gdb) detach
  
```

```sh
  
Detaching from program: target:/opt/demo/main, process 86099
[Inferior 1 (process 86099) detached]
  
```

- Quit.
```sh
  
(gdb) quit
  
```


## 2.3. Debug Running Process
- List processes under service.
```sh
  
$ systemd-cgls
  
```

```sh
  
├─demo.service
  │ ├─86095 /usr/bin/gdbserver localhost:5555 /opt/demo/main
  │ └─86099 /opt/demo/main
  
```

- Attach gdb
```sh
  
$ gdb -q -p `pidof /opt/demo/main`
  
```

- Show backtrace.
```sh
  
(gdb) bt
  
```

```c
  
#0  0x00007ffff7ceca7a in clock_nanosleep () from /lib/x86_64-linux-gnu/libc.so.6
#1  0x00007ffff7cf9a27 in nanosleep () from /lib/x86_64-linux-gnu/libc.so.6
#2  0x00007ffff7d0ec63 in sleep () from /lib/x86_64-linux-gnu/libc.so.6
#3  0x0000555555555201 in main (argc=1, argv=0x7fffffffec78) at main.c:15
  
```


# References
- [sourceware.org/gdb/current/onlinedocs/gdb.html/Server.html](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Server.html)
- [sourceware.org/gdb/current/onlinedocs/gdb.html/Connecting.html](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Connecting.html)
