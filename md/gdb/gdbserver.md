---
title: "GDB: GDBServer"
---


## Launch Program
Launch a proram under gdbserver control.
```sh
    gdb connection program args...
```

- *connection*: ip:port (TCP), TTY device, stdio, '-'.

<br>

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

<br>

## Attach to Running Process
Attach gdbserver to a running process.
```sh
    gdb --attach connection pid
```

<br>

Sample: Server attachs to running vim process. Client connect to debug.

:::::::::::::: {.columns}
::: {.column width=60%}

Server
```sh
vim
gdbserver --attach localhost:12345 `pidof vim`
  
```

:::
::: {.column}

Client
```sh
(gdb) file vim
(gdb) target remote localhost:12345
  
```

:::
::::::::::::::


<br>

## Multi Mode
Start gdbsever in multi mode
```sh
    gdbserver --multi connection
```

<br>

Sample: Server starts in multi mode. Client connects and attach to a running process.

:::::::::::::: {.columns}
::: {.column width=50%}

Server
```sh
vim
gdbserver --multi localhost:12345
  
```

:::
::: {.column}

Client
```sh
(gdb) target extended-remote localhost:12345
(gdb) attach 19832

(gdb) monitor exit
  
```

:::
::::::::::::::
