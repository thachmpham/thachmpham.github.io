---
title:  Print C++ STL Containers With GDB
---

# 1. Introduction
Looking at C++ STL containers in GDB can be confusing - they're full of internal details. With libstdc++'s Python pretty printers, GDB can show these containers in a much cleaner way. This guide will show you how to set them up and use them to make debugging easier.

# 2. Setup stdc++ Python Pretty Printers
Check if libstdc++ python available.
```sh
  
$ ls /usr/share/gcc/python/libstdcxx/
  
```

If not available, clone gcc source code.
```sh
  
$ wget https://github.com/gcc-mirror/gcc/archive/refs/heads/master.zip -O gcc.zip
$ unzip gcc.zip
  
```

Edit ~/.gdbinit to add libstdc++ python to sys.path.
```python
  
python
import sys
sys.path.insert(0, '/root/gcc-master/libstdc++-v3/python/libstdcxx')
from libstdcxx.v6.printers import register_libstdcxx_printers
register_libstdcxx_printers (None)
end
  
```

Check if libstdc++ pretty-printer added successfully.
```sh
  
$ gdb -batch -ex 'info pretty-printer'
  
```
```sh
  
global pretty-printers:
  builtin
    mpx_bound128
  libstdc++-v6
  
```


# References
- [How to pretty-print STL containers in GDB?](https://stackoverflow.com/questions/11606048/how-to-pretty-print-stl-containers-in-gdb)
- [STL Support Tools](https://sourceware.org/gdb/wiki/STLSupport)