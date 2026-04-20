---
title: 'Debug SystemD Program'
---

# Program

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

2. /etc/systemd/system/demo.service.
```sh
Description=Demo Service
ExecStart=/opt/demo/control.sh start
ExecStop=/opt/demo/control.sh stop
Type=forking
```

:::
::: {.column}

3. /opt/demo/control.sh.
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

:::
::::::::::::::

# Debug

:::::::::::::: {.columns}
::: {.column}

1. Modify start() to launch the program with gdbserver.
```sh
start() {
    start-stop-daemon --start --background \
        --make-pidfile --pidfile /var/run/demo.pid \
        --exec /usr/bin/gdbserver localhost:5555 /opt/demo/main
}
```

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

:::
::: {.column}

4. After gdbserver started, gdb automatically attached.
```sh
(gdb) info inferiors
  Num  Description       Connection                         Executable        
* 1    process 86099     1 (extended-remote localhost:5555) target:/opt/demo/main

(gdb) break main
Breakpoint 1 at 0x55555555519c: file main.c, line 7.

(gdb) continue
Continuing.
Breakpoint 1, main (argc=1, argv=0x7fffffffec78) at main.c:7
7               openlog("main", LOG_PID | LOG_CONS, LOG_USER);
```

5. Detach.
```sh
(gdb) detach
Detaching from program: target:/opt/demo/main, process 86099
[Inferior 1 (process 86099) detached]
```

:::
::::::::::::::