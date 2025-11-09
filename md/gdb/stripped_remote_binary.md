---
title: "GDB: Stripped Remote Binary"
subtitle: "Debug Stripped Binaries on a Remote Target"
---


## Local Binary


:::::::::::::: {.columns}
::: {.column width=60%}

If the remote binary is stripped, we can start GDB with a local unstripped binary.

Sample.

```sh
#--------------------
gcc -g demo.c -o demo
objcopy --only-keep-debug demo demo.debuginfo
strip --strip-debug demo
#--------------------
  
```

:::
::: {.column width=40%}

```c {.numberLines}
#include <unistd.h>

int main()
{
  int s = 0;
  int i = 0;
  while (1)
  {
    s += i;
    i += 1;
    sleep(3);
  }
}
  
```

:::
::::::::::::::


:::::::::::::: {.columns}
::: {.column width=30%}

Server.
```sh
gdbserver :12345 demo
  
```

:::
::: {.column width=60%}

Client.
```sh
$ cp demo demo_with_debug
$ objcopy --add-gnu-debuglink=demo.debuginfo demo_with_debug

(gdb) file demo_with_debug
(gdb) target remote :12345
(gdb) continue

```
:::
::::::::::::::