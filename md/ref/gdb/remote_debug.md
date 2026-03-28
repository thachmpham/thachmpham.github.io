---
title: "GDB Remote Debug"
---


# 1. Usages
## 1.1. Launch Program
Launch a proram under gdbserver control.
```sh
    gdb connection program args...
```

- *connection*: ip:port (TCP), TTY device, stdio, '-'.

Sample: Server starts ls program. Client connect to debug.

:::::::::::::: {.columns}
::: {.column}

Server
```sh
gdbserver localhost:12345 ls
  
```

:::
::: {.column}

Client
```sh
(gdb) file ls
(gdb) target remote localhost:12345
  
```

:::
::::::::::::::

## 1.2. Attach to Running Process
Attach gdbserver to a running process.
```sh
    gdb --attach connection pid
```

Sample: Server attachs to running vim process. Client connect to debug.

:::::::::::::: {.columns}
::: {.column width=60%}

Server
```sh
vim
gdbserver --attach :12345 19832
  
```

:::
::: {.column}

Client
```sh
(gdb) file vim
(gdb) target remote :12345
  
```

:::
::::::::::::::

## 1.3. Multi Mode
Start gdbsever in multi mode
```sh
    gdbserver --multi connection
```

Sample: Server starts in multi mode.
```sh
vim
gdbserver --multi :12345
  
```
:::::::::::::: {.columns}
::: {.column width=50%}

Client runs vim program.
```sh
(gdb) target extended-remote :12345
(gdb) set remote exec-file vim
(gdb) start
(gdb) monitor exit
  
```

:::
::: {.column}


Client attachs to a running process.
```sh
(gdb) target extended-remote :12345
(gdb) attach 19832
(gdb) monitor exit
  
```

:::
::::::::::::::


# 2. Applications
## 2.1. Debug Program Started By SystemD
### 2.1.1. Program

:::::::::::::: {.columns}
::: {.column}

File /opt/demo/main.c.
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

:::
::: {.column}

File /opt/demo/control.sh.
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

File /etc/systemd/system/demo.service.
```sh
Description=Demo Service
ExecStart=/opt/demo/control.sh start
ExecStop=/opt/demo/control.sh stop
Type=forking
```

:::
::::::::::::::

### 2.1.2. Debug

Modify start() to launch the program with gdbserver.
```sh
start() {
    start-stop-daemon --start --background \
        --make-pidfile --pidfile /var/run/demo.pid \
        --exec /usr/bin/gdbserver localhost:5555 /opt/demo/main
}
```

:::::::::::::: {.columns}
::: {.column}

Start gdb
```sh
$ gdb
(gdb) set tcp auto-retry on
(gdb) set tcp connect-timeout unlimited
(gdb) target remote localhost:5555
```

- Start service.
```sh
$ systemctl start demo
```

- After gdbserver started, gdb automatically attached. Show inferiors.
```sh
(gdb) info inferiors
  Num  Description       Connection                         Executable        
* 1    process 86099     1 (extended-remote localhost:5555) target:/opt/demo/main
```

:::
::: {.column}

- Set breakpoints.
```sh
(gdb) break main
Breakpoint 1 at 0x55555555519c: file main.c, line 7.

(gdb) continue
Continuing.
Breakpoint 1, main (argc=1, argv=0x7fffffffec78) at main.c:7
7               openlog("main", LOG_PID | LOG_CONS, LOG_USER);
```

- Detach.
```sh
(gdb) detach
Detaching from program: target:/opt/demo/main, process 86099
[Inferior 1 (process 86099) detached]
```

:::
::::::::::::::