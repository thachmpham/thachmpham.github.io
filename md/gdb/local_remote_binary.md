---
title: "GDB: Local & Remote Binary"
---


## Stripped Remote Binary


:::::::::::::: {.columns}
::: {.column width=60%}

If the remote binary is stripped, you can start GDB with the local unstripped version. The debug symbols from the local binary will then be loaded into GDB.

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

<br>

## Sysroot

Sysroot is the directy where gdb can find the shared libraries. Using local sysroot helps to prevent the need to fetch libraries from the remote target.

Set an alternative sysroot.
```sh
(gdb) set sysroot path
(gdb) show sysroot
```

<br>

## Solib Directory
If a library not found in sysroot, gdb then searches in solib directories.

Set solib path.
```sh
(gdb) set solib-search-prefix path
(gdb) show solib-search-prefix
```