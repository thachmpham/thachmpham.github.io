---
title: "GDB: Writing a Pretty-Printer"
subtitle: "*Print Complex Data in GDB with Python Pretty Printer*"
---

GDB Python pretty printers let you display complex C/C++ data structures in a readable format, making debugging easier and clearer. In this post, we will write a simple plugin to pretty-print an IP address **struct in_addr**.


# 1. Printer
- Create file in_addr_printer.py.
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
        return "IP address: " + s

def in_addr_lookup(val):
    typename = val.type.tag
    if typename == "in_addr":
        return InAddrPrinter(val)
    return None


gdb.printing.register_pretty_printer(
    obj=None, printer=in_addr_lookup, replace=True)
  
```


# 2. Program
- Create file main.c
```c
  
#include <stdio.h>
#include <arpa/inet.h>

int main() {
    struct in_addr address;
    inet_pton(AF_INET, "192.168.1.1", &address);

    while (1) {}

    return 0;
}
  
```

- Build.
```sh
  
$ gcc -g -o main main.c
  
```


# 3. Use Printer
- Run program under gdb.
```sh
  
$ gdb main
  
```

- Load printer. 
```sh
  
(gdb) source in_addr_printer.py

(gdb) info pretty-printer
global pretty-printers:
  builtin
    mpx_bound128
  in_addr_lookup
  
```

- Start
```sh
  
(gdb) run
^C
Program received signal SIGINT, Interrupt.
main () at main.c:8
8           while (1) {}
  
```

- Print in_addr data.
```sh
  
gdb) print address
$3 = IP address: 192.168.1.1.
  
```


# References
- [Extending GDB using Python](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Python.html#Python)
- [GDB Python modules](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Python-modules.html#Python-modules)
- [GDB.Python Testsuite](https://github.com/bminor/binutils-gdb/tree/master/gdb/testsuite/gdb.python)