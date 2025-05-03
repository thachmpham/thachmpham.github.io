---
title: Inspect ELF Files
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
hello: ELF 64-bit LSB pie executable, x86-64, dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, not stripped
  
```

- List sections.
```sh
  
$ objdump --section-headers hello
Sections:
Idx Name          Size      VMA               LMA               File off  Algn
                CONTENTS, ALLOC, LOAD, READONLY, DATA
15 .text        00000107  0000000000001060  0000000000001060  00001060  2**4
                CONTENTS, ALLOC, LOAD, READONLY, CODE
17 .rodata      00000011  0000000000002000  0000000000002000  00002000  2**2
                CONTENTS, ALLOC, LOAD, READONLY, DATA                  
24 .data        00000010  0000000000004000  0000000000004000  00003000  2**3
                CONTENTS, ALLOC, LOAD, DATA
25 .bss         00000008  0000000000004010  0000000000004010  00003010  2**0
                ALLOC                                
  
```

- Content of a section.
```sh
  
$ objdump --full-contents --section=.rodata hello
Contents of section .rodata:
 2000 01000200 68656c6c 6f2c2077 6f726c64  ....hello, world
 2010 00
  
```

- Disassemble a section.
```sh
  
$ objdump --disassemble --section=.text hello
  
```

- Symbol table.
```sh
  
$ objdump --syms hello
SYMBOL TABLE:
Start Address                   Length
0000000000000000 l    df *ABS*  0000000000000000              hello.c
0000000000000000       F *UND*  0000000000000000              puts@GLIBC_2.2.5
0000000000001149 g     F .text  000000000000001e              main
  
```

- Disassemble a function.
```sh
  
$ objdump --disassemble=main hello
0000000000001149 <main>:
    1149:       f3 0f 1e fa             endbr64
    114d:       48 83 ec 08             sub    $0x8,%rsp
    1151:       48 8d 3d ac 0e 00 00    lea    0xeac(%rip),%rdi        # 2004 <_IO_stdin_used+0x4>
    1158:       e8 f3 fe ff ff          call   1050 <puts@plt>
    115d:       b8 00 00 00 00          mov    $0x0,%eax
    1162:       48 83 c4 08             add    $0x8,%rsp
    1166:       c3                      ret
  
```

- Disassemble by addresses.
```sh
  
# Start address of main:    0x0000000000001149
# Stop address of main:     0x0000000000001149 + 0x000000000000001e = 0x1167

$ objdump --disassemble --start-address=0x0000000000001149 --stop-address=0x1167 hello
0000000000001149 <main>:
    1149:       f3 0f 1e fa             endbr64
    114d:       48 83 ec 08             sub    $0x8,%rsp
    1151:       48 8d 3d ac 0e 00 00    lea    0xeac(%rip),%rdi        # 2004 <_IO_stdin_used+0x4>
    1158:       e8 f3 fe ff ff          call   1050 <puts@plt>
    115d:       b8 00 00 00 00          mov    $0x0,%eax
    1162:       48 83 c4 08             add    $0x8,%rsp
    1166:       c3
  
```

# 2. 

# 2. Static Library
- File add.c
```c
  
int add(int a, int b)
{
    return a + b;
}
  
```

- File sub.c
```c
  
int sub(int a, int b)
{
    return a - b;
}
  
```




# References
```sh
  
$ man elf
$ man ascii
$ man objdump
  
```