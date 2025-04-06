---
title: BashDB Manual
---

## 1. Start Program Under BashDB
- Start a program.
```sh
  
$ bashdb /usr/bin/ldd
  
```

- Start & redirect output to another terminal.
```sh
  
term1$ tty
/dev/pts/1
  
```
```sh
  
term0$ bashdb /usr/bin/ldd > /dev/pts/1
  
```


## 2. Hardcode Breakpoint At A Line
```sh
  
source /opt/bashdb/share/bashdb/bashdb-trace -L /opt/bashdb/share/bashdb
_Dbg_debugger
  
```

# References
- [bashdb.sourceforge.net](https://bashdb.sourceforge.net/)
- [bashdb.readthedocs.io](https://bashdb.readthedocs.io)