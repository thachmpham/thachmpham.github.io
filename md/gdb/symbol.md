---
title: "GDB: Symbols"
---


## Data Type
:::::::::::::: {.columns}
::: {.column width=50%}

Name of data type.
```sh
  whatis expr
```

Definition of data type.
```sh
  ptype [/flags] expr
```

- *expr*: variable, struct, class name.
- *flag*: r, m, M, t, T, x, d, o (offset).


Find data type match a regex.
```sh
  info types [-q] [regex]
```

:::
::: {.column width=50%}

```cpp
class Point
{
public:
  int x, y;
  Point(int a, int b)  {}
  void move(int dx, int dy){}
};

int main() {
  Point p(1, 2);
  p.move(3, 4);
}
  
```

:::
::::::::::::::

Sample: Inspect a variable and a class.

```sh
(gdb) whatis p
type = Point
(gdb) ptype /o class Point
/* offset      |    size */  type = class Point {
                             public:
/*      0      |       4 */    int x;
/*      4      |       4 */    int y;

                               /* total size (bytes):    8 */
                             }
(gdb) info types -q Poi*
File demo.cpp:
1:      Point;
  
```

<br>

## Function
Function names matches a regex.
```sh
  info functions [-q] [-n] [-t type_regex] [regex]
```

- *-q*: quite.
- *-n*: not print non-debug symbols.
- *-t*: only print functions whose type match type_regex. Check type by *whatis*.

:::::::::::::: {.columns}
::: {.column width=50%}

```c
int echo(int n)
{
  return 0;
}

char* echo(char* s)
{
  return 0;
}
  
```

:::
::: {.column width=50%}

```sh
(gdb) info function echo
File demo.cpp:
2:      char *echo(char*);
1:      int echo(int);

(gdb) info function -t char* echo
File demo.cpp:
2:      char *echo(char*)

(gdb) info function -t int echo
File demo.cpp:
1:      int echo(int);
  
```

:::
::::::::::::::


## Variables
Global and static variables.
```sh
  info variables [-q] [-n] [-t type_regexp] [regexp]
```

<br>

## Address of Symbol
Find address of a symbol.
```sh
  info address sym
```

:::::::::::::: {.columns}
::: {.column width=50%}

```c
int g = 0;

int sum(int a, int b)
{
    return a + b;
}
  
```

:::
::: {.column width=50%}

```sh
(gdb) info address g
Symbol "g" is static storage at address 0x420020.

(gdb) info address sum
Symbol "sum" is a function at address 0x40066c.
  
```

:::
::::::::::::::

<br>

## Symbol at Address

:::::::::::::: {.columns}
::: {.column width=50%}

Find symbol at an address. Only global, static scope.
```sh
  info sym addr
```

:::
::: {.column width=50%}

```sh
(gdb) info symbol 0x420020
g in section .bss

(gdb) info symbol 0x40066c
sum in section .text
  
```

:::
::::::::::::::

<br>

## Demangle

:::::::::::::: {.columns}
::: {.column width=50%}

Demangle a name.
```sh
  demangle [--] name
  set print demangle [on|off]
  
```

- `--`: useful when name begins with a dash.

:::
::: {.column width=50%}

```sh
(gdb) demangle _ZN5PointC2Eii
Point::Point(int, int)

(gdb) demangle _ZN5Point4moveEii
Point::move(int, int)
  
```

:::
::::::::::::::

<br>

## Line Info
Print source line info.
```sh
  info line locspec
```

- *locspec*: function, address, etc.

<br>

Sample: Find line info of function, offset and instruction address.

:::::::::::::: {.columns}
::: {.column width=45%}

```c {.numberLines}
int triple(int a)
{
    return 3 * a;
}

int main()
{

}
  
```

:::
::: {.column width=55%}

```sh
(gdb) info line triple
Line 2 of "demo.c" starts at address 0x40066c <triple>
   and ends at 0x400674 <triple+8>.
(gdb) info line *triple+8
Line 3 of "demo.c" starts at address 0x400674 <triple+8>
   and ends at 0x400684 <triple+24>.
(gdb) info line *0x400674
Line 3 of "demo.c" starts at address 0x400674 <triple+8>
   and ends at 0x400684 <triple+24>.
  
```

:::
::::::::::::::