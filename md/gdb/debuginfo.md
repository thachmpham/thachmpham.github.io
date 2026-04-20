---
title: "GDB: DebugInfo"
---

# Common Procedures

:::::::::::::: {.columns}
::: {.column width=50%}

1. Split debug info.
```sh
$ objcopy --only-keep-debug demo demo.debuginfo

$ strip --strip-debug demo
```

2. Check results.
```sh
$ objdump --section-headers demo
```

3. Add debug info.
```sh
$ objcopy --add-gnu-debuglink=demo.debuginfo demo
```

:::
::: {.column width=50%}

- File demo.c
```c
int sum(int a, int b)
{
    return a + b;
}

int main()
{
    sum(1, 2);
}
```

- Build.
```sh
$ gcc -g demo.c -o demo
```

:::
::::::::::::::