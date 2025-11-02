---
title: "GDB - Print Variables"
---

## Print

:::::::::::::: {.columns}
::: {.column width=50%}
By default, GDB prints a value according to its data type. However, we can change the output format.

- d(dec), u(unsigned), f(float).
- x(hex), z(padding+hex), t(bin), o(oct).
- c(char), s(string).
- a(addr).
- r(raw, bypass pretty printer).
- array@n : n elements of array.

```c
int main(int argc, char** argv)
{
    int n = 65;
    char* s = "hello";
    return 0;
}
```

:::
::: {.column width=50%}

```python
(gdb) p n
$1 = 65
(gdb) p/x n
$2 = 0x41
(gdb) p/z n
$3 = 0x00000041
(gdb) p/t n
$4 = 1000001
(gdb) p/a &n
$6 = 0xffffffffec9c

(gdb) print s
$6 = 0x4006c0 "hello"
(gdb) p *s@3
$7 = "hel"
(gdb) p/x *s@3
$8 = {0x68, 0x65, 0x6c}
```

:::
::::::::::::::

