---
title: "GDB Watchpoint"
subtitle: "Stop when value of an expression changes"
---


| Command               | Explain                                            |
|:----------------------|:---------------------------------------------------|
| `watch p`             | Consider p as expression. Watch value of the expression.  |
| `watch *p`            | Consider *p as expression. Watch value of the expression. |
| `watch -l p`          | Find memory address of p. Watch content at the address (memory watchpoint) |
| `watch -l *p`         | Find address p points to. Watch content at the address (memory watchpoint) |
| `watch *((int *) 0xffffffffec9c)` | Watch absolute memory. |
| `rwatch p` | Stop when p is read (read watchpoint). |
| `awatch p` | Stop when p is either read or written (read-write watchpoint) . |

<br>

* * * * *

### Sample 1: watch p

:::::::::::::: {.columns}
::: {.column width=50%}

```c {.numberLines}
int main()
{
    int *p = 0; // watch p
    int a = 1;
    int b = 2;  
    p = &a;     // hit
    p = &b;     // hit
    return 0;
}
```

```sh
(gdb) start
(gdb) watch p
```

:::
::: {.column width=50%}

```sh
(gdb) continue
Hardware watchpoint 2: p
Old value = (int *) 0x0
New value = (int *) 0xffffffffec94
main () at demo.c:7

(gdb) continue
Hardware watchpoint 2: p
Old value = (int *) 0xffffffffec94
New value = (int *) 0xffffffffec90
main () at demo.c:8
```

:::
::::::::::::::

* * * * *

### Sample 2: watch *p

:::::::::::::: {.columns}
::: {.column width=50%}

```c {.numberLines}
int main()
{
  int *p = 0; //watch *p
  int a = 1;
  int b = 2;
  p = &a;     //hit: *p = 1
  *p = 10;    //hit: *p = 10
  a = 100;    //hit: *p = a = 100
  p = &b;     //hit: *p = b = 2
  return 0;
}
```

```sh
(gdb) start
(gdb) watch *p
```

:::
::: {.column width=50%}

```sh
(gdb) continue
Hardware watchpoint 2: *p
Old value = <unreadable>
New value = 1
main () at demo.c:7

(gdb) continue
Hardware watchpoint 2: *p
Old value = 1
New value = 10
main () at demo.c:8

(gdb) continue
Hardware watchpoint 2: *p
Old value = 10
New value = 100
main () at demo.c:9

(gdb) continue
Hardware watchpoint 2: *p
Old value = 100
New value = 2
main () at demo.c:10  
```

:::
::::::::::::::

* * * * *


### Sample 3: Memory Watchpoint

:::::::::::::: {.columns}
::: {.column width=60%}

```c {.numberLines}
int main()
{  
  int a = 1, b = 2;
  int *p = &a;    // watch -l *p
  p = &b;
  b = 20;
  *p = 200;
  a = 10;         // hit
  a = 100;        // hit
  return 0;
}
```

Watch the memory p points to.

```sh
(gdb) start
(gdb) watch -l *p
```

:::
::: {.column width=40%}

```python
(gdb) continue
Hardware watchpoint 2: -location *p
Old value = 1
New value = 10
main () at demo.c:9

(gdb) continue
Hardware watchpoint 2: -location *p
Old value = 10
New value = 100
main () at demo.c:10
```

:::
::::::::::::::

* * * * *


### Sample 4: Watch Absolute Memory.

:::::::::::::: {.columns}
::: {.column width=50%}

```c {.numberLines}
int main()
{
  int n = 3;  // watch *((int*)address of n)
  n += 1;     // hit
  return 0;
}
```

:::
::: {.column width=50%}

```python
(gdb) print &n
$1 = (int *) 0xffffffffec9c
(gdb) print *((int *) 0xffffffffec9c)
$2 = 3

# watch a 4-byte region at the address
(gdb) watch *((int *) 0xffffffffec9c)

(gdb) continue
Hardware watchpoint 2: *((int *) 0xffffffffec9c)
Old value = 3
New value = 4
main () at demo.c:5
```

:::
::::::::::::::
  
* * * * *

