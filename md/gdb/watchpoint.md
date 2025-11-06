---
title: "GDB: Watchpoints"
---


## Watchpoints
Stop when the value of the expression changes.
```sh
  watch [-l|-location] expression
```

- `expression`: a variable, an address cast to an data type, a C/C++ expression.
- `-l, -location`: watch the memory refrred to by the expression.

Sample: watch p (*stop when p changed*).

:::::::::::::: {.columns}
::: {.column width=40%}

```c {.numberLines}
int main()
{
  int *p = 0;
  int a = 1;
  int b = 2;  
  p = &a;
  p = &b;
  return 0;
}
  
```

:::
::: {.column width=60%}

```python
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

Sample: watch \*p (*stop when \*p changed*).

:::::::::::::: {.columns}
::: {.column width=50%}

```c {.numberLines}
int main()
{
  int *p = 0; // *p = undef
  int a = 1;
  int b = 2;
  p = &a; // *p = 1
  *p = 10;// *p = 10
  a = 100;// *p = a = 100
  p = &b; // *p = b = 2
  return 0;
}
  
```

:::
::: {.column width=50%}

```python
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
