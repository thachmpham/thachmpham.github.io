---
title: BashDB Manual
---

# 1. Start
Start a script under bashdb control.
```sh
  
$ bashdb /usr/bin/ldd
  
```

# 2. Redirect Output
Redirect output to another terminal.

- Get path of terminal 1.
```sh
  
term1$ tty
/dev/pts/1
  
```

- From terminal 0, redirect output to terminal 1.
```sh
  
term0$ bashdb /usr/bin/ldd > /dev/pts/1
  
```


## 2. Hardcode Breakpoint At A Line
- Add the below code to the bash script.
```sh
  
source /opt/bashdb/share/bashdb/bashdb-trace -L /opt/bashdb/share/bashdb
_Dbg_debugger
  
```

# References
- [bashdb.sourceforge.net](https://bashdb.sourceforge.net/)
- [bashdb.readthedocs.io](https://bashdb.readthedocs.io)