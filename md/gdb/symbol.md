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