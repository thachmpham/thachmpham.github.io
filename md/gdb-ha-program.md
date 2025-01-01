---
title: 'GDB'
subtitle: '**Debug A High Availability Program**'
---


# 1. Introduction
High availability systems are designed for continuous service, typically relying on multi-threaded programs where some threads manage real-time tasks while others handle external events. Debugging these systems is more challenging because interrupting or pausing the program to investigate issues can disrupt service and affect system reliability.


# 2. Lab
## 2.1. High Availability Program
In this lab, we will the debug the below OpenSAF AMF program.

- Source code: [github.com/thachmpham/samples](https://github.com/thachmpham/samples/tree/main/opensaf/worker_thread)

- File main.c
```c
  
int main(int argc, char ** argv)
{
    //...    
    saAmfInitialize_o4(&amfHandler, &callbacks, &apiVersion);
    
    saAmfComponentRegister(amfHandler, &name, 0);

    pthread_create(&thread, NULL, threadFunction, NULL);

    saAmfSelectionObjectGet(amfHandler, &monitorFd);    
    struct pollfd fds[1];
    fds[0].fd = monitorFd;
    fds[0].events = POLLIN;
    
    while (poll(fds, 1, -1) != -1)
    {
        if (fds[0].revents & POLLIN)
        {            
            saAmfDispatch(amfHandler, SA_DISPATCH_ONE);
        }
    }
}

void* threadFunction(void* arg)
{
    trace();
    for (int i = 0; ; i++)
	{
        trace("continue... i=%d\n", i);
        sleep(5);
    }
}
  
```

- File control.sh.
```sh
  
start() {
    logger -st demo 'start'
    start-stop-daemon --start --background \
        --make-pidfile --pidfile /var/run/demo.pid \
        --exec /opt/demo/main
}

stop() {
    logger -st demo 'stop'
    start-stop-daemon --stop --pidfile /var/run/demo.pid
}
  
```

- Build and install.
```sh
  
$ mkdir /opt/demo
$ cp worker_thread/* /opt/demo
$ cd /opt/demo
$ ./build.sh
$ systemctl start opensafd
$ ./import-imm.sh
$ amf-state su all safSu=SC-1,safSg=demo,safApp=demo
  
```

We will debug the program in two phases:

- Debug the entrypoint when the program started (Section 2.2).
- Debug the program while it is running (Section 2.3).


## 2.2. Debug Entrypoint
When debugging entrypoint of a AMF application, we encounters the below points:

- The program starts as daemon via the command configured by attribute `saAmfCtRelPathInstantiateCmd` of class `SaAmfCompType`.
- If the AMF operation cannot finished within `saAmfCtDefClcCliTimeout` of class `SaAmfCompType`, AMF will assume that the operation failed and stop the operation.
- If the component does not register within `saAmfCompInstantiateTimeout` of class `SaAmfComp`, AMF will automatically kill and restart the process.

First, we adjust these timeouts to have enough time for debugging.

- Adjust saAmfCtDefClcCliTimeout. Time unit is nanosecond.
```sh
  
$ immcfg -a saAmfCtDefClcCliTimeout=600000000000 safVersion=1,safCompType=demo
  
```

- Modify function start() in script control.sh to launch the program with gdbserver.
```sh
  
start() {
    logger -st demo 'start'
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

- Connect and wait for gdbserver.
```sh
  
(gdb) target remote localhost:5555
  
```

- Instantiate service unit, which will start gdbserver.
```sh
  
$ amf-adm unlock-in safSu=SC-1,safSg=demo,safApp=demo
  
```

- After gdbserver started, gdb automatically attached.
```sh
  
# console log
Remote debugging using localhost:5555
Reading /opt/demo/main from remote target...
  
```

- Show inferiors.
```sh
  
(gdb) info inferiors
  
```

```sh
  
# console log
  Num  Description       Connection                Executable        
* 1    process 379783    1 (remote localhost:5555) target:/opt/demo/main
  
```

- Set breakpoints.
```sh
  
(gdb) break main
  
```

```sh
  
# console log
Breakpoint 1 at 0xaaaaaaaa0b64: file main.c, line 40.
  
```

- Continue.
```sh
  
(gdb) continue
  
```

```sh
  
# console log
Continuing.

Breakpoint 1, main (argc=1, argv=0xfffffffffba8) at main.c:40
40          openlog("demo", LOG_PID, LOG_USER);
  
```


- Continue.
```sh
  
(gdb) continue
  
```

```sh
  
# syslog
SC-1 demo[379783]: main: start
SC-1 demo[379783]: threadFunction: 
SC-1 demo[379783]: threadFunction: continue... i=0
SC-1 demo[379783]: threadFunction: continue... i=1
SC-1 demo[379783]: threadFunction: continue... i=2
SC-1 demo[379783]: threadFunction: continue... i=3
  
```

- Detach and quit.
```sh
  
(gdb) detach
(gdb) quit
  
```


## 2.2. Debug Running Program
When debugging entrypoint of a AMF application, we encounters the below points:





# References
- [GDB Connecting to a Remote Target](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Connecting.html)
- [GDB Non-Stop Mode](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Non_002dStop-Mode.html)
