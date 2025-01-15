---
title: BashDB Manual
---

# 1. Start
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

# References
- [bashdb.sourceforge.net](https://bashdb.sourceforge.net/)
- [bashdb.readthedocs.io](https://bashdb.readthedocs.io)