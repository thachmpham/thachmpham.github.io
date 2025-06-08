---
title: "GDB: Writing a Pretty-Printer"
subtitle: "*Print Complex Data in GDB with Python Pretty Printer*"
---

GDB Python pretty printers let you display complex C/C++ data structures in a readable format, making debugging easier and clearer. In this post, we will write simple plugins to print **in_addr** and **linked list** structs.


# 1. IP Address Printer
In this section, we will write a plugin to print IP address (`struct in_addr`).
```c
  
// Header:  /usr/include/netinet/in.h

/* Internet address.  */
typedef uint32_t in_addr_t;
struct in_addr
  {
    in_addr_t s_addr;
  };
  
```

## 1.1. Program
```c
  
#include <arpa/inet.h>

int main() {
    struct in_addr address;
    inet_pton(AF_INET, "192.168.1.1", &address);

    while (1) {} // pause for debug
}
  
```

```sh
  
$ gcc -g -o main main.c
  
```

## 1.2. Printer
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
    typename = val.type.name
    if typename == "in_addr":
        return InAddrPrinter(val)
    return None

gdb.printing.register_pretty_printer(
    obj=None, printer=in_addr_lookup, replace=True)
  
```


## 1.3. Run
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

- Run and pause to debug.
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


# 2. Linked List Printer
In this section, we will write a plugin to print a linked list.

## 2.1. Program
```c
  
struct Node {
    int data;
    struct Node* next;
};

int main() {    
    struct Node* first = (struct Node*)malloc(sizeof(struct Node));
    struct Node* second = (struct Node*)malloc(sizeof(struct Node));
    struct Node* third = (struct Node*)malloc(sizeof(struct Node));
    
    first->data = 1;
    first->next = second;

    second->data = 2;
    second->next = third;

    third->data = 3;
    third->next = NULL;
    
    while (1) {} // pause for debug    
}
  
```

```sh
  
$ gcc -g -o main main.c
  
```

## 2.2. Printer
- Create file in_addr_printer.py.
```python
  
import gdb

class NodePrinter:
    def __init__(self, val):
        self.val = val

    def to_string(self):
        s = "\n"
        node = self.val

        while node != 0:
            data = node["data"]

            current_address = node.const_value()

            next = node["next"]
            next_address = "null" if next == 0 else next.const_value()

            s += f"Node {current_address} [data={data}, next={next_address}]\n"

            node = next
        # end while

        return s

def linked_list_lookup(val):
    if str(val.type) == "struct Node *":
        return NodePrinter(val)
    return None

gdb.printing.register_pretty_printer(
    obj=None, printer=linked_list_lookup, replace=True)
  
```


## 2.3. Run
- Run program under gdb.
```sh
  
$ gdb main
  
```

- Load printer. 
```sh
  
(gdb) source linked_list_printer.py

(gdb) info pretty-printer 
global pretty-printers:
  builtin
    mpx_bound128
  linked_list_lookup
  
```

- Run and pause to debug.
```sh
  
(gdb) run
Starting program: /root/demo/main 
^C
Program received signal SIGINT, Interrupt.
main () at main.c:23
23          while (1) {} // pause for debug
  
```

- Print linked list node.
```sh
  
(gdb) print third
$1 = 
Node 0x5555555592e0 [data=3, next=null]

(gdb) print second
$2 = 
Node 0x5555555592c0 [data=2, next=0x5555555592e0]
Node 0x5555555592e0 [data=3, next=null]

(gdb) print first
$3 = 
Node 0x5555555592a0 [data=1, next=0x5555555592c0]
Node 0x5555555592c0 [data=2, next=0x5555555592e0]
Node 0x5555555592e0 [data=3, next=null]
  
```


# References
- [Extending GDB using Python](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Python.html#Python)
- [GDB.Python Testsuite](https://github.com/bminor/binutils-gdb/tree/master/gdb/testsuite/gdb.python) + grep register_pretty_printers
- [gdb.printing](https://sourceware.org/gdb/current/onlinedocs/gdb.html/gdb_002eprinting.html#gdb_002eprinting) + /usr/share/gdb/python/gdb/printing.py
- [gdb.types](https://sourceware.org/gdb/current/onlinedocs/gdb.html/gdb_002etypes.html#gdb_002etypes) + /usr/share/gdb/python/gdb/types.py
- [gdb.value](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Values-From-Inferior.html#Values-From-Inferior)