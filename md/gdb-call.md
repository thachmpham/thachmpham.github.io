---
title: Execute Functions With GDB Call
---

# 1. Introduction
The call command in GDB allows you to execute functions, modify variables, and evaluate expressions during debugging. It helps test functions, change program states, and interact with the program dynamically without restarting execution.

# 2. Lab
## 2.1. Function Without Arguments
```c++
  
void print_hello()
{
    printf("hello\n");
}
  
```

```sh
  
(gdb) call print_hello()
hello
  
```


## 2.2. Pass By Value
```c++
  
void print_int(int a)
{
    printf("%d\n", a);
}
  
```

```sh
  
(gdb) call print_int(3)
3
  
```


## 2.3. Pass By Reference
```c++
  
void print_ref_int(int &a)
{
    printf("%d\n", a);
}
  
```

```sh
  
(gdb) set $ptr = (int*) malloc(sizeof(int))
(gdb) set *$ptr = 3
(gdb) call print_ref_int(*$ptr)
3
  
```


## 2.3. Pass By Pointer
```c++
  
void print_pointer_int(int *a)
{
    printf("%d\n", *a);
}
  
```

```sh
  
(gdb) set $ptr = (int*) malloc(sizeof(int))
(gdb) set *$ptr = 3
(gdb) call print_ref_int(*$ptr)
3
  
```


# References
- [GDB Call](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Calling.html)
- [GDB Pass By Reference](https://stackoverflow.com/questions/10460567/cannot-call-function-with-reference-parameter-in-gdb)