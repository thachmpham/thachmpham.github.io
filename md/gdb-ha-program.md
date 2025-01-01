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
2024-12-31T23:46:23.560527-05:00 SC-1 demo: start
2024-12-31T23:46:23.563433-05:00 SC-1 demo[384582]: main: start
2024-12-31T23:46:23.565075-05:00 SC-1 demo[384582]: main: receive poll event
2024-12-31T23:46:23.565192-05:00 SC-1 demo[384582]: healthcheckCallback: count = 0
2024-12-31T23:46:23.565281-05:00 SC-1 demo[384582]: threadFunction: 
2024-12-31T23:46:23.565324-05:00 SC-1 demo[384582]: threadFunction: continue... i=0
2024-12-31T23:46:28.566365-05:00 SC-1 demo[384582]: threadFunction: continue... i=1
2024-12-31T23:46:33.566979-05:00 SC-1 demo[384582]: threadFunction: continue... i=2
2024-12-31T23:46:33.652628-05:00 SC-1 demo[384582]: main: receive poll event
2024-12-31T23:46:33.652835-05:00 SC-1 demo[384582]: healthcheckCallback: count = 1
2024-12-31T23:46:38.568911-05:00 SC-1 demo[384582]: threadFunction: continue... i=3
2024-12-31T23:46:43.569081-05:00 SC-1 demo[384582]: threadFunction: continue... i=4
  
```

- Detach and quit.
```sh
  
(gdb) detach
(gdb) quit
  
```


## 2.2. Debug Running Process
After each healthcheck timeout, AMF sends a ping to the running process. If there is no response, AMF assumes the process is down and triggers failover. Below is list of healthcheck timeouts:

- Attribute `saAmfHctDefPeriod` in class `SaAmfHealthcheckType`.
- Attribute `saAmfHealthcheckPeriod` in class `SaAmfHealthcheck`.

For convinience, we can adjust these timeouts to have more time for debugging.

- Adjust saAmfHctDefPeriod. Time unit is nanosecond.
```sh
  
$ immcfg -a saAmfHctDefPeriod=600000000000 safHealthcheckKey=demo,safVersion=1,safCompType=demo
  
```

The healthcheck ping will be handled by some threads of the AMF program, while other threads manage different tasks. In the below output, threads 1, 2, 3 handle ping requests; while thread 4 executes function threadFunction() - a while loop.

```sh
  
$ gdb -q -p `pidof /opt/demo/main` -batch -ex 'thread apply all backtrace'
  
```

```c++
  
// console log
Thread 4
#0  __GI___clock_nanosleep ()   at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:48
#1  __GI___nanosleep ()         at ../sysdeps/unix/sysv/linux/nanosleep.c:25
#2  __sleep ()                  at ../sysdeps/posix/sleep.c:55
#3  threadFunction ()           at main.c:157
#4  start_thread ()             at ./nptl/pthread_create.c:442
#5  thread_start ()             at ../sysdeps/unix/sysv/linux/aarch64/clone.S:79

Thread 3
#0  __GI___poll ()              at ../sysdeps/unix/sysv/linux/poll.c:41
#1  poll ()                     at /usr/include/aarch64-linux-gnu/bits/poll2.h:39
#2  mdtm_process_recv_events_tcp () at src/mds/mds_dt_trans.c:986
#3  start_thread ()             at ./nptl/pthread_create.c:442
#4  thread_start ()             at ../sysdeps/unix/sysv/linux/aarch64/clone.S:79

Thread 2
#0  __GI___poll ()              at ../sysdeps/unix/sysv/linux/poll.c:41
#1 poll ()                      at /usr/include/aarch64-linux-gnu/bits/poll2.h:39
#2  osaf_poll_no_timeout ()     at src/base/osaf_poll.c:31
#3  osaf_poll ()                at src/base/osaf_poll.c:44
#4  osaf_poll_one_fd ()         at src/base/osaf_poll.c:133
#5  ncs_tmr_wait ()             at ./src/base/timer/timer_handle.h:76
#6  start_thread ()             at ./nptl/pthread_create.c:442
#7  thread_start ()             at ../sysdeps/unix/sysv/linux/aarch64/clone.S:79

Thread 1
#0  __GI___poll ()              at ../sysdeps/unix/sysv/linux/poll.c:41
#1  main ()                     at main.c:95
  
```

We will use gdb non-stop mode to keep thread 1,2,3 running while pause thread 4 for debugging.

- Start gdb, enable non-stop.
```sh
  
$ gdb
(gdb) set non-stop on  
(gdb) set pagination off
(gdb) set confirm off
  
```

- Attach.
```sh
  
(gdb) shell pidof main
(gdb) attach 403184
  
```

```sh
  
# console log
Attaching to process 403184
Thread 2 "main" stopped.
Thread 3 "main" stopped.
Thread 4 "main" stopped.
  
```

- Continue all in background.
```sh
  
(gdb) continue -a &
  
```

- All threads are running.
```sh
  
(gdb) info threads
  
```

```sh
  
# console log
  Id   Target Id                                 Frame 
* 1    Thread 0xffff929a4440 (LWP 403184) "main" (running)
  2    Thread 0xffff923af120 (LWP 403185) "main" (running)
  3    Thread 0xffff9237f120 (LWP 403186) "main" (running)
  4    Thread 0xffff9234f120 (LWP 403187) "main" (running)
  
```

- Go to thread 4.
```sh
  
(gdb) thread 4
  
```

```sh
  
# console log
[Switching to thread 4 (Thread 0xffff9234f120 (LWP 403187))](running)
  
```

- Interrupt thread 4.
```sh
  
(gdb) interrupt
  
```

```sh
  
# console log
Thread 4 "main" stopped.
  
```

- Thread 4 is interrupted, while others are running.
```sh
  
(gdb) info threads
  
```

```sh
  
# console log
  Id   Target Id                                 Frame 
  1    Thread 0xffff929a4440 (LWP 403184) "main" (running)
  2    Thread 0xffff923af120 (LWP 403185) "main" (running)
  3    Thread 0xffff9237f120 (LWP 403186) "main" (running)
* 4    Thread 0xffff9234f120 (LWP 403187) "main" __GI___clock_nanosleep () at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:48
  
```

- Show backtrace.
```sh
  
(gdb) backtrace
  
```

```c++
  
// console log
#0  __GI___clock_nanosleep ()   at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:48
#1  __GI___nanosleep ()         at ../sysdeps/unix/sysv/linux/nanosleep.c:25
#2  __sleep ()                  at ../sysdeps/posix/sleep.c:55
#3  threadFunction ()           at main.c:157
#4  start_thread ()             at ./nptl/pthread_create.c:442
#5  thread_start ()             at ../sysdeps/unix/sysv/linux/aarch64/clone.S:79
  
```


# 3. Cheatsheets
## 3.1. Debug Entrypoint
- GDB script.
```sh
  
set tcp auto-retry on
set tcp connect-timeout unlimited
  
```

- GDB.
```sh
  
gdb -q -x debug.gdb -ex 'target remote localhost:5555'
  
```

- GDBServer.
```sh
  
gdbserver localhost:5555 /opt/demo/main
  
```

## 3.2. Debug Running Process
- GDB script.
```sh
  
set non-stop on
set pagination off
set confirm off
  
```

- GDB.
```sh
  
gdb -x debug.gdb -ex "attach $(pidof /opt/demo/main)"
  
```

```sh
  
(gdb) continue -a &
(gdb) thread 4
(gdb) interrupt
  
```

# References
- [GDB Connecting to a Remote Target](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Connecting.html)
- [GDB Non-Stop Mode](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Non_002dStop-Mode.html)
- [GDB Background Execution](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Background-Execution.html)