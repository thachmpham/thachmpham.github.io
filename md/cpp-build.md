---
title: Build & Inspect C/C++ Code
---

# 1. Hello World
- File hello.c.
```c
  
#include <stdio.h>

int main() {
    printf("hello, world\n");
}
  
```

- Build.
```sh
  
$ gcc -Og -o hello hello.c
  
```

- File information.
```sh
  
$ file hello
  
```

- List dependencies.
```sh
  
$ ldd hello
  
```

- List sections.
```sh
  
$ objdump --section-headers hello
  
```
