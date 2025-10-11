---
title: GDB Cheatsheet
---


### Configuration
```sh
  
# .gdbearlyinit
set startup-quitely on
  
```

```sh
  
# .gdbinit  
set pagination off
set confirm off
  
```


### Environment Variables
```sh
  
set environment myvar=val
unset environment myvar
show environment myvar
  
```


### Command-line Arguments
```sh
  
gdb --args program arg1 arg2...
(gdb) run arg1 arg2...
(gdb) set args arg1 arg2...
  
```


### Attach
```sh
  
gdb -p pid
gdb -p pid -x script -batch

gdb -ex "attach pid"
gdb -ex "attach pid" -ex "set non-stop on"
  
```
  