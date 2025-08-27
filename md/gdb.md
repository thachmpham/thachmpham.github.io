---
title: GDB Cheatsheet
---


# Configuration
## Startup Setup
- File: `~/.gdbearlyinit`
```sh
  
set startup-quitely on
  
```

- File: `~/.gdbinit`
```sh
  
set pagination off
set confirm off
  
```

## Environment Variables
```sh
  
set environment VAR=value
unset environment VAR

show environment VAR
  
```


# Launch
## Command-line Arguments
```sh

$ gdb --args program arg1 arg2...

```

```sh
  
$ gdb program
(gdb) run arg1 arg2...
  
```

```sh
  
$ gdb program
(gdb) set args arg1 arg2...
(gdb) run
  
```


# Attach
- Normal attach.
```sh
  
$ gdb --pid=<pid>
$ gdb -p <pid>
$ gdb -ex "attach <pid>"
  
```

- Attach with non-stop mode. Useful for multi-thread debug.
```sh
  
$ gdb -ex "set non-stop on" -ex "attach <pid>"
  
```

- Attach in batch mode. Useful for quick query.
```sh
  
$ gdb -batch -p <pid>
$ gdb -batch -p <pid> -x <gdb script>
  
```