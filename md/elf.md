---
title: Inspect ELF Files
---


The ELF (Executable and Linkable Format) is the standard file format for programs on Linux. It has three main types:

- *Relocatable Object File:* Contains binary code and data that can be combined with other relocatable files at link time to create an executable.
- *Executable Object File:* Contains binary code and data ready to be loaded into memory and run directly by the operating system.
- *Shared Object File:* A special type of relocatable file that can be loaded and linked dynamically, either when the program starts or while it’s running.


# 1. Relocatable Object File
- File add.c
```c
  
int add(int a, int b)
{
    return a + b;
}
  
```

- Build.
```sh
  
$ gcc -Og -c add.c
  
```

- File info.
```sh
  
$ file add.o
add.o: ELF 64-bit LSB relocatable, x86-64
  
```

- Symbol table.
```sh
  
$ objdump --syms add.o
SYMBOL TABLE:
Start Address                   Length
0000000000000000 g     F .text  0000000000000008 add
  
```

- Disassemble.
```sh
  
$ objdump --disassemble add.o
0000000000000000 <add>:
   0:   f3 0f 1e fa             endbr64
   4:   8d 04 37                lea    (%rdi,%rsi,1),%eax
   7:   c3                      ret
  
```


# 2. Executable Object File
- File main.c
```c
  
int add(int a, int b);

int main(int argc, char** agrv)
{
    add(1, 2);

    return 0;
}
  
```

- Build
```sh
  
$ gcc -Og -c add.c
$ gcc -Og -c main.c
$ gcc -Og -o prog main.o add.o
  
```

- File info.
```sh
  
$ file prog
prog: ELF 64-bit LSB pie executable, x86-64, dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2
  
```

- Symbol table.
```sh
  
$ objdump --syms prog
000000000000114a g     F .text  0000000000000008              add
0000000000001129 g     F .text  0000000000000021              main
  
```

- Disassemble.
```sh
  
$ objdump --disassemble prog
0000000000001129 <main>:
    1129:       f3 0f 1e fa             endbr64
    112d:       48 83 ec 08             sub    $0x8,%rsp
    1131:       be 02 00 00 00          mov    $0x2,%esi
    1136:       bf 01 00 00 00          mov    $0x1,%edi
    113b:       e8 0a 00 00 00          call   114a <add>
    1140:       b8 00 00 00 00          mov    $0x0,%eax
    1145:       48 83 c4 08             add    $0x8,%rsp
    1149:       c3                      ret

000000000000114a <add>:
    114a:       f3 0f 1e fa             endbr64
    114e:       8d 04 37                lea    (%rdi,%rsi,1),%eax
    1151:       c3                      ret
  
```


# 3. Shared Object File
- File sub.c.
```c
  
int sub(int a, int b)
{
    return a - b;
}
  
```

- File main.c.
```c

int sub(int a, int b);

int main(int argc, char** agrv)
{
    sub(1, 2);

    return 0;
}

```

- Build shared object file.
```sh
  
$ gcc -Og -fPIC -c sub.c
$ gcc -shared -o libSub.so sub.o
  
```

- Build executable object file.
```sh
  
$ gcc -Og -c main.c
$ gcc -Og -o prog main.o -L. -lSub
  
```

- File info.
```sh
  
$ file libSub.so
libSub.so: ELF 64-bit LSB shared object, x86-64, dynamically linked
  
```

- Symbol table.
```sh
  
$ objdump --syms libSub.so
00000000000010f9 g     F .text  0000000000000009 subtract

$ objdump --syms prog
0000000000001149 g     F .text  0000000000000021              main
0000000000000000       F *UND*  0000000000000000              subtract
  
```

- Disassemble.
```sh
  
$ objdump --disassemble prog
Disassembly of section .plt.sec:
0000000000001050 <subtract@plt>:
    1050:       f3 0f 1e fa             endbr64
    1054:       ff 25 76 2f 00 00       jmp    *0x2f76(%rip)        # 3fd0 <subtract@Base>
    105a:       66 0f 1f 44 00 00       nopw   0x0(%rax,%rax,1)

Disassembly of section .text:
0000000000001149 <main>:
    1149:       f3 0f 1e fa             endbr64
    114d:       48 83 ec 08             sub    $0x8,%rsp
    1151:       be 02 00 00 00          mov    $0x2,%esi
    1156:       bf 01 00 00 00          mov    $0x1,%edi
    115b:       e8 f0 fe ff ff          call   1050 <subtract@plt>
    1160:       b8 00 00 00 00          mov    $0x0,%eax
    1165:       48 83 c4 08             add    $0x8,%rsp
    1169:       c3                      ret
  
```

`subtract@plt` is a function in the Procedure Linkage Table (PLT) that handles dynamic linking to the actual `subtract` function in a shared library. When the program first calls `subtract@plt`, it resolves and jumps to the real `subtract` function. After the first call, subsequent calls directly invoke `subtract`.


- Run.
```sh
  
$ export LD_LIBRARY_PATH=.

$ gdb prog
  
```

```sh
  
(gdb) start
Temporary breakpoint 1, 0x0000555555555149 in main ()


(gdb) info sharedlibrary
From                To                  Syms Read   Shared Object Library
0x00007ffff7fc6000  0x00007ffff7ff0195  Yes         /lib64/ld-linux-x86-64.so.2
0x00007ffff7fb9040  0x00007ffff7fb9102  Yes (*)     ./libSub.so
0x00007ffff7c28800  0x00007ffff7dafcb9  Yes         /lib/x86_64-linux-gnu/libc.so.6


(gdb) info functions subtract
Non-debugging symbols:
0x0000555555555050  subtract@plt
0x00007ffff7fb90f9  subtract


(gdb) break subtract@plt
Breakpoint 2 at 0x555555555050


(gdb) break subtract
Breakpoint 3 at 0x7ffff7fb90f9


(gdb) c
Continuing.
Breakpoint 2, 0x0000555555555050 in subtract@plt ()


(gdb) c
Continuing.
Breakpoint 3, 0x00007ffff7fb90f9 in subtract () from ./libSub.so
  
```


# References
- [Computer Systems: A Programmer's Perspective, Randal E. Bryant, David R. O'Hallaron](https://csapp.cs.cmu.edu/)
- [Executable and Linking Format](https://man7.org/linux/man-pages/man5/elf.5.html)