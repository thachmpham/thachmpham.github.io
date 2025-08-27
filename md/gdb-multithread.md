---
title: GDB & Multiple Threads
---


# 1. Thread Control
- Display info of all threads.
```sh
  
info threads
  
```

- Display info of current thread.
```sh
  
thread
  
```

- Switch to a thread.
```sh
  
thread <tid>
  
```

- Apply a command to threads.
```sh
  
thread apply <tids> command

thread apply all command
  
```

- Enable thread debug.
```sh
  
set debug threads [on|off]
  
```
When set to 'on', GDB prints extra messages whenever threads are created or destroyed. If you press Ctrl+C to interrupt the target process, the process is not terminated immediately. Instead, the SIGINT is delivered to GDB, allowing you to decide whether to resume or terminate the target process.


# 2. Non-stop Mode
## 2.1 Enable & Disable
- In non-stop mode, you can pause some threads while keep other threads continue to run freely.
```sh
  
set non-stop [on|off]
show non-stop
  
```

- If using the CLI, pagination breaks non-stop. Can try the below workaround.
```sh
  
set pagination off
set non-stop on
  
```
- Attach to a running thread with non-stop enable.
```sh
  
$ gdb -ex "set non-stop on" -ex "attach <pid>"
  
```

## 2.2 Continue & Interrupt
- Continue threads. If non-stop mode is enabled, the `continue` command only the current thread. To continue all threads, use the -a option.
```sh
  
continue
continue -a
  
```

- Interrupt threads. If non-stop mode is enabled, interrupt only the current thread. To interrupt all threads, use the -a option.
```sh
  
interrupt
interrupt -a
  
```


# 3. Asynchronous Execution
GDB has two ways to run programs: synchronous (foreground) and asynchronous (background).

- Synchronous: GDB waits until the program stops before showing the prompt.
- Asyncronous: GDB immediately gives a command prompt so that you can issue other commands while your program runs.

Synchronous execution is especially useful in conjunction with non-stop mode for debugging programs with multiple threads.

- Continue Synchronous.
```sh
  
continue &
continue -a &
  
```


# 4. Lab
- File: demo.c
```
  
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

#define NUM_THREADS 4

void* child_thread(void* arg) {
    while (1) {
        printf("Child thread, tid 0x%lx\n", pthread_self());
        sleep(7);
    }
    return NULL;
}

int main() {
    pthread_t tids[NUM_THREADS];
    for (int i = 0; i < NUM_THREADS; i++)
        pthread_create(&tids[i], NULL, child_thread, NULL);

    while (1) {
        printf("Main thread, tid %lu\n", pthread_self());
        sleep(11);
    }
    return 0;
}
  
```

- Build.
```sh
  
$ gcc -g -o demo demo.c -pthread
  
```

- Run.
```sh
  
./demo
  
```

- Attach GDB.
```sh
  
$ gdb -ex "set non-stop on" -ex "attach `pidof demo`"
  
```

- Check non-stop status.
```sh
  
(gdb) show non-stop 
Controlling the inferior in non-stop mode is on.
  
```

- Resume all threads. Add the & option to get the prompt right after the command.
```sh
  
(gdb) continue -a &

(gdb) info thread
  Id   Target Id                                Frame 
* 1    Thread 0x7f5705c93740 (LWP 33038) "demo" (running)
  2    Thread 0x7f5705c92640 (LWP 33039) "demo" (running)
  3    Thread 0x7f5705491640 (LWP 33040) "demo" (running)
  4    Thread 0x7f5704c90640 (LWP 33041) "demo" (running)
  5    Thread 0x7f570448f640 (LWP 33042) "demo" (running)
  
```

- Interrupt a thread (tid = 3), while keep other threads running.
```sh
  
(gdb) thread 3
[Switching to thread 3 (Thread 0x7f5705491640 (LWP 33040))](running)

(gdb) interrupt
Thread 3 "demo" stopped.
0x00007f5705d7b7f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0, req=req@entry=0x7f5705490e00, rem=rem@entry=0x7f5705490e00) at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
78      in ../sysdeps/unix/sysv/linux/clock_nanosleep.c

(gdb) info thread
  Id   Target Id                                Frame 
  1    Thread 0x7f5705c93740 (LWP 33038) "demo" (running)
  2    Thread 0x7f5705c92640 (LWP 33039) "demo" (running)
* 3    Thread 0x7f5705491640 (LWP 33040) "demo" 0x00007f5705d7b7f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0, req=req@entry=0x7f5705490e00, rem=rem@entry=0x7f5705490e00) at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
  4    Thread 0x7f5704c90640 (LWP 33041) "demo" (running)
  5    Thread 0x7f570448f640 (LWP 33042) "demo" (running)
  
```

- Resume the interrupted thread.
```sh
  
(gdb) thread 3
[Switching to thread 3 (Thread 0x7f5705491640 (LWP 33040))]
#0  0x00007f5705d7b7f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0, req=req@entry=0x7f5705490e00, rem=rem@entry=0x7f5705490e00) at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
78      in ../sysdeps/unix/sysv/linux/clock_nanosleep.c

(gdb) continue &
Continuing.
  
```

- Interrupt all threads, then resume only one thread (tid == 2).
```sh
  
(gdb) interrupt -a
Thread 3 "demo" stopped.
0x00007f5705d7b7f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0, req=req@entry=0x7f5705490e00, rem=rem@entry=0x7f5705490e00) at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
78      in ../sysdeps/unix/sysv/linux/clock_nanosleep.c

Thread 5 "demo" stopped.
0x00007f5705d7b7f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0, req=req@entry=0x7f570448ee00, rem=rem@entry=0x7f570448ee00) at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
78      in ../sysdeps/unix/sysv/linux/clock_nanosleep.c

Thread 1 "demo" stopped.
0x00007f5705d7b7f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0, req=req@entry=0x7fff0bbb3d70, rem=rem@entry=0x7fff0bbb3d70) at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
78      in ../sysdeps/unix/sysv/linux/clock_nanosleep.c

Thread 4 "demo" stopped.
0x00007f5705d7b7f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0, req=req@entry=0x7f5704c8fe00, rem=rem@entry=0x7f5704c8fe00) at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
78      in ../sysdeps/unix/sysv/linux/clock_nanosleep.c

Thread 2 "demo" stopped.
0x00007f5705d7b7f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0, req=req@entry=0x7f5705c91e00, rem=rem@entry=0x7f5705c91e00) at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
78      in ../sysdeps/unix/sysv/linux/clock_nanosleep.c

(gdb) thread 2
[Switching to thread 2 (Thread 0x7f5705c92640 (LWP 33039))]
#0  0x00007f5705d7b7f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0, req=req@entry=0x7f5705c91e00, rem=rem@entry=0x7f5705c91e00) at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
78      in ../sysdeps/unix/sysv/linux/clock_nanosleep.c

(gdb) continue &
Continuing.

(gdb) info threads
  Id   Target Id                                Frame 
  1    Thread 0x7f5705c93740 (LWP 33038) "demo" 0x00007f5705d7b7f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0, req=req@entry=0x7fff0bbb3d70, rem=rem@entry=0x7fff0bbb3d70) at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
* 2    Thread 0x7f5705c92640 (LWP 33039) "demo" (running)
  3    Thread 0x7f5705491640 (LWP 33040) "demo" 0x00007f5705d7b7f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0, req=req@entry=0x7f5705490e00, rem=rem@entry=0x7f5705490e00) at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
  4    Thread 0x7f5704c90640 (LWP 33041) "demo" 0x00007f5705d7b7f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0, req=req@entry=0x7f5704c8fe00, rem=rem@entry=0x7f5704c8fe00) at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
  5    Thread 0x7f570448f640 (LWP 33042) "demo" 0x00007f5705d7b7f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0, req=req@entry=0x7f570448ee00, rem=rem@entry=0x7f570448ee00) at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
  
```


# References
- (gdb) help thread
- (gdb) help show non-stop
- (gdb) help continue
- (gbd) help interrupt
- (gdb) apropos thread
- (gdb) apropos non-stop