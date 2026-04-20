---
title: 'GDB Remote Debug'
---

# Single Mode
## Launch Program

:::::::::::::: {.columns}
::: {.column width=50%}

Server
```sh
$ gdbserver localhost:12345 ls
```

:::
::: {.column width=50%}

Client
```sh
(gdb) file ls
(gdb) target remote localhost:12345
```

:::
::::::::::::::

## Attach to Running Process
:::::::::::::: {.columns}
::: {.column width=50%}

Server
```sh
$ vi
$ gdbserver --attach localhost:12345 `pidof vi`
```

:::
::: {.column width=50%}

Client
```sh
(gdb) file vi
(gdb) target remote localhost:12345
```

:::
::::::::::::::

# Multi Mode
## Launch Program

:::::::::::::: {.columns}
::: {.column width=50%}

Server.
```sh
$ gdbserver --multi :12345
```

:::
::: {.column width=50%}

Client.
```sh
(gdb) target extended-remote :12345
(gdb) set remote exec-file vi
(gdb) start
(gdb) monitor exit
```

:::
::::::::::::::


## Attach to Running Process


:::::::::::::: {.columns}
::: {.column width=50%}

Server.
```sh
$ vi
$ gdbserver --multi localhost:12345
```

:::
::: {.column width=50%}

Client.
```sh
(gdb) target extended-remote localhost:12345
(gdb) attach 19832  # pidof vi
(gdb) monitor exit
```

:::
::::::::::::::

# Debug Program Started By SystemD
## Program

:::::::::::::: {.columns}
::: {.column}

1. /opt/demo/main.c.
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

```sh
$ gcc -g main.c -o main
```

:::
::: {.column}

2. /opt/demo/control.sh.
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

3. /etc/systemd/system/demo.service.
```sh
Description=Demo Service
ExecStart=/opt/demo/control.sh start
ExecStop=/opt/demo/control.sh stop
Type=forking
```

:::
::::::::::::::

## Procedure
1. Modify start() to launch the program with gdbserver.
```sh
start() {
    start-stop-daemon --start --background \
        --make-pidfile --pidfile /var/run/demo.pid \
        --exec /usr/bin/gdbserver localhost:5555 /opt/demo/main
}
```

:::::::::::::: {.columns}
::: {.column}

2. Start gdb.
```sh
$ gdb
(gdb) set tcp auto-retry on
(gdb) set tcp connect-timeout unlimited
(gdb) target remote localhost:5555
```

3. Start service.
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

4. Set breakpoints.
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