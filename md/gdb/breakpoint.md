---
title: "GDB: Breakpoints"
---


## Set Breakpoints
:::::::::::::: {.columns}
::: {.column width=50%}

A breakpoint makes your program stop whenever a certain point in the program is reached.
```sh
break [-qualified] locspec
```
- **locspec**: can be line number, function name, address.
- **qualified**: full match.

```c {.numberLines}
int sum(int a, int b)
{
  return a + b;
}

int main()
{
  int a = 1, b = 2;
  int s = sum(a, b);
}
```

:::
::: {.column width=50%}

```sh
(gdb) b 3
(gdb) b sum
(gdb) b demo.c:3
(gdb) b demo.c:sum

(gdb) b -qualified exit

(gdb) disassemble sum
   0x000000000040066c <+0>:     sub     sp, sp, #0x10
   0x0000000000400670 <+4>:     str     w0, [sp, #12]
   0x0000000000400674 <+8>:     str     w1, [sp, #8]
   0x0000000000400678 <+12>:    ldr     w1, [sp, #12]
   0x000000000040067c <+16>:    ldr     w0, [sp, #8]
   0x0000000000400680 <+20>:    add     w0, w1, w0
   0x0000000000400684 <+24>:    add     sp, sp, #0x10
   0x0000000000400688 <+28>:    ret

(gdb) b *sum+4
(gdb) b 0x0000000000400670
```

:::
::::::::::::::
