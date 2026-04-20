---
title: 'GDB: Keep Child Process Under GDB Control'
---

:::::::::::::: {.columns}
::: {.column width=50%}

1. File main.c
```sh
#include <unistd.h>

int main()
{
    fork();
    while (1) {}
}
```

```sh
$ gcc -g main.c -o main
```

:::
::: {.column width=50%}

2. Configure gdb.
```sh
(gdb) set detach-on-fork off
(gdb) catch fork
```

3. Run debug.
```sh
(gdb) run
Catchpoint 1 (forked process 4097), arch_fork (ctid=0x7ffff7d85a10)

(gdb) next
[New inferior 2 (process 4097)]

(gdb) info inferiors 
  Num  Description       Connection           Executable        
* 1    process 4094      1 (native)           /root/demo/demo   
  2    process 4097      1 (native)           /root/demo/demo
```

:::
::::::::::::::