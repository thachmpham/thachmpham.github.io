---
title: "GDB: Breakpoints"
---


## Set Breakpoints
:::::::::::::: {.columns}
::: {.column width=50%}

Stop program whenever a point in the program is reached.
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

<br>

## Conditional Breakpoints
Set condition.
```sh
break locspec [-force-condition] if cond
```

- **cond**: only stop at breakpoint when cond is true.
- **force-condition**: force to enable to cond.

Change condition.
```sh
condition breakpoint [cond]
```

<br>

## Regex Breakpoints
:::::::::::::: {.columns}
::: {.column width=50%}

```sh
rbeak regex
rbreak file:regex
```

- **regex**: standard regex, like grep.

:::
::: {.column width=50%}

```sh
rbreak sum.*
rbreak demo.c:sum.*

# all functions in current program
rbreak .
# all functions in file
rbreak demo.c:.
```

:::
::::::::::::::

<br>

## Thread-Specific Breakpoints
```sh
break locspec thread thread-id [if cond]
```

<br>

## Inferior-Specific Breakpoints
```sh
break locspec inferior inferior-id [if cond]
```

<br>

## Temporary Breakpoints
Automatically deleted after the hit.
```sh
tbreak args
```

<br>

## Commands
:::::::::::::: {.columns}
::: {.column width=60%}

Set commands.
```sh
commands [breakpoints...]
    commands...
end
```

Change commands.
```sh
commands breakpoints...
    commands...
end
```

:::
::: {.column width=40%}

```sh
break sum
commands
bt
c
end

break sum
break main
commands 1, 2
bt
c
end
```

:::
::::::::::::::

<br>

## Manage Breakpoints
Show breakpoints
```sh
info break
```

Delete the breakpoints.
```sh
delete [breakpoints] [list…]
```

Enable, disable breakpoints
```sh
enable breakpoints [breakpoints]
disable breakpoints [breakpoints]
```

Delete all breakpoints at to locspec.
```sh
clear locspec
```

Save, restore breakpoints.
```sh
save breakpoints file
source file
```