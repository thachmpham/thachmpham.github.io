---
title: "LD_PRELOAD"
subtitle: "*Make a Shared Library Do Something Different*"
---

**`LD_PRELOAD`** is an environment variable that allows to load a shared library before any other libraries when a program starts.
This gives us the ability to override standard functions with custom implementations without modifiying the original program.

**`LD_DEBUG`** is an environment varibale used to trace and debug the dynamic linker during program startup.
```sh
  
LD_PRELOAD="lib_01.so" program
LD_PRELOAD="lib_01.so:lib_02.so" program

LD_DEBUG=all LD_PRELOAD="lib_01.so:lib_02.so" program

man ld.so
  
```

# 1. Override malloc
- File main.c
```c
  
#include <stdlib.h>

int main(int argc, char** argv)
{
    char *p = malloc(8);
    free(p);

    return 0;
}
  
```

- Build
```sh
  
$ gcc -g -o main main.c
  
```

- File mymalloc.c
```c
  
#define _GNU_SOURCE

#include <stdio.h>
#include <dlfcn.h>

static void* (*real_malloc)(size_t) = NULL;

static void mtrace_init(void)
{
    real_malloc = dlsym(RTLD_NEXT, "malloc");
    if (NULL == real_malloc)
    {
        fprintf(stdout, "dlsym malloc failed: %s\n", dlerror());
    }
}

void* malloc(size_t size)
{
    if(real_malloc == NULL)
    {
        mtrace_init();
    }

    fprintf(stdout, "malloc(%ld) = ", size);
    void *p = real_malloc(size);
    fprintf(stdout, "%p\n", p);
    return p;
}
  
```

- Build
```sh
  
$ gcc -g -o mymalloc.so -fPIC -shared mymalloc.c -ldl
  
```

- Run program normally.
```sh
  
$ ./main
  
```

- Run with preload library.
```sh
  
$ LD_PRELOAD=./mymalloc.so ./main

malloc(8) = 0x593c4106e2a0
  
```

- Enable linker log.
```sh
  
$ LD_DEBUG=all LD_PRELOAD=./mymalloc.so ./main

file=./mymalloc.so [0];  needed by ./main [0]
file=./mymalloc.so [0];  generating link map
  dynamic: 0x00007272bc74cdf8  base: 0x00007272bc749000   size: 0x0000000000004030
    entry: 0x00007272bc749000  phdr: 0x00007272bc749040  phnum:                 11

file=libc.so.6 [0];  needed by ./main [0]
file=libc.so.6 [0];  generating link map
  dynamic: 0x00007272bc602940  base: 0x00007272bc400000   size: 0x0000000000211d90
    entry: 0x00007272bc42a390  phdr: 0x00007272bc400040  phnum:                 14

relocation processing: /lib/x86_64-linux-gnu/libc.so.6
symbol=malloc;  lookup in file=./main [0]
symbol=malloc;  lookup in file=./mymalloc.so [0]
binding file /lib/x86_64-linux-gnu/libc.so.6 [0] to ./mymalloc.so [0]: normal symbol `malloc' [GLIBC_2.2.5]

relocation processing: ./mymalloc.so (lazy)
symbol=malloc;  lookup in file=./main [0]
symbol=malloc;  lookup in file=./mymalloc.so [0]
binding file ./main [0] to ./mymalloc.so [0]: normal symbol `malloc' [GLIBC_2.2.5]

calling init: /lib/x86_64-linux-gnu/libc.so.6
calling init: ./mymalloc.so

initialize program: ./main
transferring control: ./main

malloc(8) = 0x60827a8082a0
  
```

# 2. Override open, read, write, close
