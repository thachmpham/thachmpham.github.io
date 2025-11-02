---
title: "GDB: Variables & Memory"
---

## Print Variables
:::::::::::::: {.columns}
::: {.column width=60%}

By default, GDB prints a value according to its data type. However, we can change the output format.

- d (dec), u (unsigned), f (float).
- x (hex), z (pad + hex), t (bin), o (oct).
- c (char), s (string).
- a (address).
- r (raw, bypass pretty printer).
- array@n : n elements of array.

```c {.numberLines}
int main()
{
  int n = 0x01020304;
  char* str = "hello";
  return 0;
}
```

:::
::: {.column width=40%}

```c
(gdb) p n
$1 = 16909060
(gdb) p/x n
$2 = 0x1020304
(gdb) p/z n
$3 = 0x1020304
(gdb) p/a &n
$6 = 0xffffffffec9c

(gdb) print str
$6 = 0x4006c0 "hello"
(gdb) p *str@3
$7 = "hel"
(gdb) p/x *str@3
$8 = {0x68, 0x65, 0x6c}
```

:::
::::::::::::::


## Convenience Variables
:::::::::::::: {.columns}
::: {.column width=60%}

GDB provides convenience variables that can hold on to a value and read later.

:::
::: {.column width=40%}

```c
(gdb) set $var1 = n
(gdb) p $var
$15 = 16909060
```

:::
::::::::::::::


## Examine Memory
The command x is used to examine memory.
```sh
    x/nuf addr
```
- **n**: Number of unit. Default: 1.
- **u**: Unit (b, h, w, g).
- **f**: Format (x, d, u, o, t, a, c, f, s, i).
- **addr**: Starting address.

```c
(gdb) x/1wd &n
0xffffffffec9c: 16909060
(gdb) x/1wx &n
0xffffffffec9c: 0x01020304
(gdb) x/4bx &n
0xffffffffec9c: 0x04    0x03    0x02    0x01

(gdb) x/s str
0x4006c8:       "hello"
(gdb) x/6bc str
0x4006c8: 104 'h' 101 'e' 108 'l' 108 'l' 111 'o' 0 '\000'
```

## Automatic Display
Automatic display variables each time the program stops.

:::::::::::::: {.columns}
::: {.column width=50%}

```c {.numberLines}
int main()
{
  int i = 0, sum = 0;
  while (i < 3)
  {
    sum += i;
    i += 1;
  }
}
```

:::
::: {.column width=50%}

```python
(gdb) set print frame-info location
(gdb) break 7

(gdb) display i
(gdb) display sum

(gdb) while 1
 >continue
 >end
```

:::
::::::::::::::

```c
Breakpoint 1, main (argc=1, argv=0xffffffffee18) at demo.c:7
1: i = 0
2: sum = 0

Breakpoint 1, main (argc=1, argv=0xffffffffee18) at demo.c:7
1: i = 1
2: sum = 1

Breakpoint 1, main (argc=1, argv=0xffffffffee18) at demo.c:7
1: i = 2
2: sum = 3
```