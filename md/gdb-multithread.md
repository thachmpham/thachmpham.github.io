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

Background execution is especially useful in conjunction with non-stop mode for debugging programs with multiple threads.

- Continue in background.
```sh
  
continue &
continue -a &
  
```