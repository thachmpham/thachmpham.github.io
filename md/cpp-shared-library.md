---
title: C/C++ Shared Library
---


# 1. Build Shared Library
- File math.c.
```c
  
int multiple(int a, int b)
{
    return a * b;
}

int divide(int a, int b)
{
    return a / b;
}
  
```

- Build.
```sh
  
$ gcc -fPIC -c math.c

$ gcc -shared -o libmath.so math.o
  
```


# 2. Link
- File main.c.
```c
  
int multiple(int a, int b);
int divide(int a, int b);

int main(int argc, char** argv)
{
    multiple(2, 4);
    divide(4, 2);
}
  
```

- Build.
```sh
  
$ gcc -c main.c

$ gcc -o prog main.o -L. -lmath
  
```

# 3. Run
```sh
  
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:.

$ ldd prog
    linux-vdso.so.1 (0x00007f2760785000)
    libmath.so (0x00007f2760775000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f2760400000)
    /lib64/ld-linux-x86-64.so.2 (0x00007f2760787000)

$ ./prog
  
```


# 4. Function Address
- Start program under gdb.
```sh
  
$ gdb prog
  
```

```sh
  
(gdb) start
Temporary breakpoint 1, 0x0000555555555171 in main ()


(gdb) info sharedlibrary
From                To                  Syms Read   Shared Object Library
0x00007ffff7fc6000  0x00007ffff7ff0195  Yes         /lib64/ld-linux-x86-64.so.2
0x00007ffff7fb9040  0x00007ffff7fb9127  Yes (*)     libmath.so
0x00007ffff7c28800  0x00007ffff7dafcb9  Yes         /lib/x86_64-linux-gnu/libc.so.6
(*): Shared library is missing debugging information.


(gdb) info address multiple
Symbol "multiple" is at 0x7ffff7fb90f9 in a file compiled without debugging.
  
```

- Distance from address of function `multiple` to start address of `libmath.so`.
```python
  
>>> 0x7ffff7fb90f9 - 0x00007ffff7fb9040
185
  
```

- Address of `.text` section in `libmath.so`.
```sh
  
$ objdump --section-headers libmath.so
Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  9 .text         000000e7  0000000000001040  0000000000001040  00001040  2**4
  
```

- Address of function `multiple` in `libmath.so`.
```sh
  
$ objdump --dynamic-syms libmath.so
DYNAMIC SYMBOL TABLE:
Start Address                   Length
0000000000001110 g    DF .text  0000000000000017 divide
00000000000010f9 g    DF .text  0000000000000017 multiple
  
```

- Distance from address of function `multiple` to start address of `.text` section.
```python
  
>>> 0x00000000000010f9 - 0x00001040
185
  
```

- **Observation**
The distance between the `multiple` function and the start of the `.text` section in `libmath.so` is equal to the distance between the `multiple` function’s address and the start address where `libmath.so` is loaded in memory at runtime. Both are 185 bytes.


