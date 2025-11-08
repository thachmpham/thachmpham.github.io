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


<br>

## Multi Mode
Start gdbsever in multi mode
```sh
    gdbserver --multi connection
```

<br>

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
