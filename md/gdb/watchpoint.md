---
title: "GDB: Watchpoints"
---


## Watchpoints
Stop when the value of the expression changes.
```sh
  watch [-l|-location] expression
```

- `expression`: a variable, an address cast to an data type, a C/C++ expression.
- `-l, -location`: watch the memory refrred to by the expression.

<br>

## Read Watchpoints

Stop when the expression is read.
```sh
  rwatch [-l|-location] expression
```

<br>

## Read-Write Watchpoints
Stop when the expression is either read or written.
```sh
  awatch [-l|-location] expression
```
