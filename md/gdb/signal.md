---
title: "GDB: Signals"
---


## Handle Signals
GDB can detect any occurrence of a signal in the program. You can specify what to do for each signal, except SIGKILL.

```sh
  handle signals [actions]
  info signals [id]
```

- *actions*:
  - stop (enter debugger), nostop.
  - print (print signal when occur), noprint.
  - pass (pass signal to program), nopass.
  

<br>

## Send Signals
Send a signal once the program resumes after hitting a breakpoint.
```sh
  signal id
  signal 0    # continue
```

Send a queue of signals once the program resumes after hitting a breakpoint.
```sh
  queue-signal id
```

Using linux command to send signals.
```sh
  kill -n signum pid
  kill -s signame pid
  kill -l
```