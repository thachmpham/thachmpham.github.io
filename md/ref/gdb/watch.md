---
title: "GDB Watchpoint"
---


## Expression Watchpoint
Stop when the value of the expression changes.
```sh
  watch expression
```

- `expression`: a variable, an address cast to an data type, a C/C++ expression.


Sample: stop when p changes.

:::::::::::::: {.columns}
::: {.column width=50%}

```c {.numberLines}
int main()
{
  int *p = 0;//watch p
  int a = 1;
  int b = 2;  
  p = &a; //hit
  p = &b; //hit
  return 0;
}
  
```

:::
::: {.column width=50%}

```python
(gdb) start

(gdb) p &a
$1 = (int *) 0xffffffffec94
(gdb) p &b
$2 = (int *) 0xffffffffec90

(gdb) watch p
Hardware watchpoint 2: p

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

Sample: stop when \*p changes.

:::::::::::::: {.columns}
::: {.column width=60%}

```c {.numberLines}
int main()
{
  int *p = 0; //watch *p
  int a = 1;
  int b = 2;
  p = &a; //hit: *p = 1
  *p = 10;//hit: *p = 10
  a = 100;//hit: *p = a = 100
  p = &b; //hit: *p = b = 2
  return 0;
}
  
```

:::
::: {.column width=40%}

```python
(gdb) start

(gdb) watch *p
Hardware watchpoint 2: *p

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


<br>

## Memory Watchpoints
Stop when the memory referred to by the expression changes.
```sh
  watch -location expression
```

- `-location, -l`: watch the memory referred to by the expression. The memory address is fixed at the moment the watchpoint is set. Stop when contents at that address change.

Sample: watch the memory p points to.

:::::::::::::: {.columns}
::: {.column width=60%}

```c {.numberLines}
int main()
{  
  int a = 1, b = 2;
  int *p = &a; // watch -l *p
  p = &b;
  b = 20;
  *p = 200;
  a = 10; // hit
  a = 100;// hit
  return 0;
}
  
```

:::
::: {.column width=40%}

```python
(gdb) watch -l *p

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


Sample: Watch an absolute memory.

:::::::::::::: {.columns}
::: {.column width=40%}

```c {.numberLines}
int main()
{
  int n = 3;// watch *((int*)address of n)
  n += 1;   // hit
  return 0;
}
  
```

:::
::: {.column width=60%}

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
  
| Command               | Explain                                            |
|:----------------------|:---------------------------------------------------|
| `watch p`             | Consider p as expression. Watch value of the expression.  |
| `watch *p`            | Consider *p as expression. Watch value of the expression. |
| `watch -l p`          | Find memory address of p. Watch content at the address.   |
| `watch -l *p`         | Find address p points to. Watch content at the address.   |
| `watch *((int*)addr)` | Watch absolute memory. |

<br>

## Read Watchpoints

Stop when the expression is read.
```sh
  rwatch [-l|-location] expression
```

<br>

## Read-Write Watchpoints
Stop when the expression is either read or written.
```sh
  awatch [-l|-location] expression
```

<br>