---
title: 'Automatically Backtrace On Termination'
---

# Backtrace on SIGSEGV

:::::::::::::: {.columns}
::: {.column width=50%}

File main.c
```c
void crash()
{
    int *p = 0;
    *p = 42;
}

int main()
{
    crash();
}
```

```sh
$ gcc -g main.c -o main
$ gdb main
```

:::
::: {.column width=50%}

1. Catch SIGSEGV and setup commands.
```sh
(gdb) catch signal SIGSEGV
(gdb) commands
>backtrace
>continue
>end
```

2. Run debug
```sh
(gdb) run
Catchpoint 1 (signal SIGSEGV)
#0  0x000000000040067c in crash () at demo.c:4
#1  0x0000000000400698 in main () at demo.c:9
Program terminated with signal SIGSEGV, Segmentation fault.
```

:::
::::::::::::::