---
title: 'GDB Remote Debug'
---

# Common Procedures
## Single Mode

:::::::::::::: {.columns}
::: {.column width=50%}

Launch a program.

```sh
# server
$ gdbserver localhost:12345 ls
```

```sh
# client
(gdb) file ls
(gdb) target remote localhost:12345
```
:::
::: {.column width=50%}

Attach to a running process.

```sh
# server
$ vi
$ gdbserver --attach localhost:12345 `pidof vi`
```

```sh
# client
(gdb) file vi
(gdb) target remote localhost:12345
```

:::
::::::::::::::


## Multi Mode


:::::::::::::: {.columns}
::: {.column width=50%}

Launch Program

```sh
# sever
$ gdbserver --multi :12345
```

```sh
# client
(gdb) target extended-remote :12345
(gdb) set remote exec-file vi
(gdb) start
(gdb) monitor exit
```

:::
::: {.column width=50%}

Attach to a running process.

```sh
# server
$ vi
$ gdbserver --multi localhost:12345
```

```sh
# client
(gdb) target extended-remote localhost:12345
(gdb) attach 19832  # pidof vi
(gdb) monitor exit
```

:::
::::::::::::::
