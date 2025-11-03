---
title: "GDB: Variables"
---

## Print Variables
:::::::::::::: {.columns}
::: {.column width=60%}

By default, GDB prints a value according to its data type. However, we can change the output format.
```sh
print [[OPTION]... --] [/FMT] [EXP]
```
FMT, format:

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
(gdb) p/a &n
$4 = 0xffffffffec9c


(gdb) p str
$1 = 0x4006c0 "hello"
(gdb) p *str@6
$2 = "hello"
(gdb) p/c *str@6
$3 = {104 'h', 101 'e', 108 'l', 108 'l', 111 'o', 0 '\000'}
(gdb) p/x *str@6
$4 = {0x68, 0x65, 0x6c, 0x6c, 0x6f, 0x0}

```

:::
::::::::::::::


## Convenience Variables
:::::::::::::: {.columns}
::: {.column width=60%}

GDB provides convenience variables that can hold on to a value and read later.
```sh
set $variable = value
show convenience
```
:::
::: {.column width=40%}

```c
int n = 16909060;

(gdb) set $var1 = n
(gdb) p $var
$15 = 16909060
```

:::
::::::::::::::


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


## Examine Memory
:::::::::::::: {.columns}
::: {.column width=50%}

The command x is used to examine memory.
```sh
    x/nuf addr
```
- **n**: Number of unit. Default: 1.
- **u**: Unit (b, h, w, g).
- **f**: Format (x, d, u, o, t, a, c, f, s, i).
- **addr**: Starting address.

:::
::: {.column width=50%}

```c

short n = 0x0102;

(gdb) x/1wd &n
0xffffffffec9c: 258
(gdb) x/1wx &n
0xffffffffec9c: 0x01020
(gdb) x/4bx &n
0xffffffffec9c: 0x02    0x01
```

:::
::::::::::::::