---
title: 'GDB'
subtitle: '**Debug A Program Started By A Systemd Service**'
---


# 1. Introduction
Debugging a program launched by systemd can be tricky, especially when trying to find the debug entrypoint of the program due to the lifecycle of the systemd service. The way systemd manages services can make debugging more challenging.


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
  
Description=Demo Service
ExecStart=/opt/demo/control.sh start
ExecStop=/opt/demo/control.sh stop
Type=forking
    
```


## 2.2. Debug.
- Modify start() functions in control.sh script to launch the program with gdbserver.
```sh
  
start() {
    start-stop-daemon --start --background \
        --make-pidfile --pidfile /var/run/demo.pid \
        --exec /usr/bin/gdbserver localhost:5555 /opt/demo/main
}
  
```

- Start gdb.
```sh
  
$ gdb
  
```

- Enable tcp auto-retry, unlimited timeout. This is useful when the program is launched in parallel with GDB.
```sh
  
(gdb) set tcp auto-retry on
(gdb) set tcp connect-timeout unlimited
  
```

- Connect and wait for gdbserver which will be started by systemctl.
```sh
  
(gdb) target remote localhost:5555
  
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

- After gdbserver started by systemctl, gdb automatically attached.
```sh
  
# console log
Remote debugging using localhost:5555
Reading /opt/demo/main from remote target...
_start () at ../sysdeps/aarch64/dl-start.S:22
  
```

- Show inferiors.
```sh
  
(gdb) info inferiors
  
```

```sh
  
# console log
  Num  Description       Connection                         Executable        
* 1    process 86099     1 (extended-remote localhost:5555) target:/opt/demo/main
  
```

- Set breakpoints.
```sh
  
(gdb) break main
  
```

```sh
  
# console log
Breakpoint 1 at 0x55555555519c: file main.c, line 7.
  
```

- Continue.
```sh
  
(gdb) continue
  
```

```sh
  
# console log
Continuing.

Breakpoint 1, main (argc=1, argv=0x7fffffffec78) at main.c:7
7               openlog("main", LOG_PID | LOG_CONS, LOG_USER);
  
```

- Continue.
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
  
# console log
Detaching from program: target:/opt/demo/main, process 86099
[Inferior 1 (process 86099) detached]
  
```

- Quit.
```sh
  
(gdb) quit
  
```


# 3. Cheatsheets
- GDB script.
```sh
  
set confirm off
set pagination off

set tcp auto-retry on
set tcp connect-timeout unlimited
  
```

- GDB.
```sh
  
gdb -q -x debug.gdb -ex "target remote localhost:5555"
  
```

- GDBServer.
```sh
  
gdbserver localhost:5555 main
  
```


# References
- [GDBServer](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Server.html)
- [GDB Connecting to a Remote Target](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Connecting.html)
