---
title: "GDB: Pretty Printers"
---


## Stdc++ Printer
:::::::::::::: {.columns}
::: {.column width=50%}
The pretty printer for C++ STL containers is usually installed into system together with g++ and can be found at the below path.

- /usr/share/gcc/python/libstdcxx

:::
::: {.column width=50%}

```python
(gdb) info pretty-printer 
  builtin
  libstdc++-v6
```

:::
::::::::::::::

If the stdc++ printer is not installed yet, you can manually install it.

- Download stdc++ printer.
```sh
wget https://github.com/gcc-mirror/gcc/archive/refs/heads/master.zip \
    -O gcc.zip

unzip gcc.zip
```

- Add to .gdbinit.
```python
python
  import sys
  from libstdcxx.v6.printers import register_libstdcxx_printers
  
  sys.path.insert(0, '/root/gcc-master/libstdc++-v3/python')
  register_libstdcxx_printers (None)
end
```

<br>

## Manage Printers
Enable, disable and show pretty printers info.
```sh
enable pretty-printer [object-regexp [name-regexp]]
disable pretty-printer [object-regexp [name-regexp]]
info pretty-printer [object-regexp [name-regexp]]
```

<br>

## Writing a Printer
:::::::::::::: {.columns}
::: {.column width=50%}

A pretty-printer consists of 3 steps:

- The printer implementation.
- A lookup function to detect if the type is supported.
- Register the lookup function.

:::
::: {.column width=50%}

```c
typedef uint32_t in_addr_t;
struct in_addr
{
    in_addr_t s_addr;
};
```

:::
::::::::::::::

**Sample: Implement Printer for in_addr.**

:::::::::::::: {.columns}
::: {.column width=50%}

**Step 1. Printer.**

```python
import gdb

class InAddrPrinter:
  def __init__(self, val):
    self.val = val
  def to_string(self):
    octets = self.val.bytes
    s = ""
    for x in octets:
      s += str(x) + "."
    return s
```

:::
::: {.column width=50%}

**Step 2. Lookup function.**

```python
import gdb

def in_addr_lookup(val):
  typename = val.type.name
  if typename == "in_addr":
    return InAddrPrinter(val)
  return None
```

:::
::::::::::::::

**Step 3. Register lookup function**.   
*File: in_addr_printer.py.*
```python
from gdb.printing import register_pretty_printer

register_pretty_printer(
  obj=None, printer=in_addr_lookup, replace=True)
```

:::::::::::::: {.columns}
::: {.column width=50%}

**Step 4. Test.**

```c
#include <arpa/inet.h>

int main()
{
  struct in_addr addr;

  inet_pton(AF_INET, 
    "192.168.1.1", 
    &addr);
}
```

:::
::: {.column width=50%}

```python


(gdb) source in_addr_printer.py

(gdb) info pretty-printer 
  builtin
  in_addr_lookup
  libstdc++-v6

(gdb) print addr
$1 = 192.168.1.1.
```

:::
::::::::::::::


## APIs
The related APIs to implement pretty printers.
```python

(gdb) pi
>>> import gdb
>>> from gdb.printing import register_pretty_printer
>>> help(register_pretty_printer)
'''
Register pretty-printer PRINTER with OBJ.
Arguments:
    obj: Either an objfile, progspace, or None (in which case the printer
        is registered globally).
    printer: Either a function of one argument (old way) or any object
        which has attributes: name, enabled, __call__.
    replace: If True replace any existing copy of the printer.
        Otherwise if the printer already exists raise an exception.
'''
```


## References
- [github/binutils-gdb/python/lib](https://github.com/bminor/binutils-gdb/tree/master/gdb/python/lib/gdb)
- [github.com/gcc/libstdc++-v3/python](https://github.com/gcc-mirror/gcc/tree/master/libstdc%2B%2B-v3/python/libstdcxx)