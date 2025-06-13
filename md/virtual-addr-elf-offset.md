---
title: Virtual Address & ELF Offset
---


In an ELF file, the offset from a instruction's address to the start of the .text section is preserved when the shared library is loaded into memory. The distance between the function’s virtual address and the base address of the loaded library at runtime is identical to the offset of the instruction from the .text section in the ELF file.

$$
instruction\_va - base\_va = instruction\_offset - text\_offset
$$

| Term               | Description                                              |
|--------------------|----------------------------------------------------------|
| `instruction_va`   | Virtual address of the instruction in memory.            |
| `base_va`          | Base virtual address where the shared library is loaded in memory |
| `instruction_offset` | Offset of the instruction in the `.text` section in the ELF file. |
| `text_offset`      | Offset of the `.text` section in the ELF file.           |


# 1. Demo Program
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

- Run
```sh
  
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:.

$ ldd prog
    linux-vdso.so.1 (0x00007f2760785000)
    libmath.so (0x00007f2760775000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f2760400000)
    /lib64/ld-linux-x86-64.so.2 (0x00007f2760787000)

$ ./prog
  
```


# 2. Function Address
Check the formula with function `multiple` of library `libmath.so`.

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

- **Observation**: The distance between the `multiple` function and the start of the `.text` section in `libmath.so` is equal to the distance between the `multiple` function’s address and the start address where `libmath.so` is loaded in memory at runtime. Both are 185 bytes.


# 3. Instruction Address
Translate the virtual address of an instruction in process memory to the offset in ELF file.

- File main_2.c.
```c
  
int multiple(int a, int b);
int divide(int a, int b);

int main(int argc, char** argv)
{
    multiple(2, 4);
    divide(4, 0);
}
  
```

- Build.
```sh
  
$ gcc -o prog_2 main_2.c -L. -lmath
  
```

- Run program under gdb.
```sh
  
$ gdb prog_2


(gdb) run
Starting program: /root/demo/prog_2
Program received signal SIGFPE, Arithmetic exception.
0x00007ffff7fb9122 in divide () from libmath.so


(gdb) disassemble 
Dump of assembler code for function divide:
   0x00007ffff7fb9110 <+0>:     endbr64
   0x00007ffff7fb9114 <+4>:     push   %rbp
   0x00007ffff7fb9115 <+5>:     mov    %rsp,%rbp
   0x00007ffff7fb9118 <+8>:     mov    %edi,-0x4(%rbp)
   0x00007ffff7fb911b <+11>:    mov    %esi,-0x8(%rbp)
   0x00007ffff7fb911e <+14>:    mov    -0x4(%rbp),%eax
   0x00007ffff7fb9121 <+17>:    cltd
=> 0x00007ffff7fb9122 <+18>:    idivl  -0x8(%rbp)
   0x00007ffff7fb9125 <+21>:    pop    %rbp
   0x00007ffff7fb9126 <+22>:    ret
End of assembler dump


(gdb) info sharedlibrary
From                To                  Syms Read   Shared Object Library
0x00007ffff7fc6000  0x00007ffff7ff0195  Yes         /lib64/ld-linux-x86-64.so.2
0x00007ffff7fb9040  0x00007ffff7fb9127  Yes (*)     libmath.so
0x00007ffff7c28800  0x00007ffff7dafcb9  Yes         /lib/x86_64-linux-gnu/libc.so.6
  
```

The process crashed while executing the instruction at address `0x00007ffff7fb9122`. Let's translate the address to offset in file `libmath.so`.

- Calculate the distance from the instruction to base address of `libmath.so` in memory.
```python
  
>>> 0x00007ffff7fb9122 - 0x00007ffff7fb9040
226
  
```

- Find offset of `.text` section in `libmath.so`.
```sh
  
$ objdump --section-headers libmath.so
Idx Name          Size      VMA               LMA               File off  Algn
  9 .text         000000e7  0000000000001040  0000000000001040  00001040  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE

# The offset of text section is 0x00001040.
  
```



- Calculate offset of the instruction in elf file.
```python
  
#   hex(text_section + distance)
>>> hex(0x00001040 + 226)
'0x1122'

# The offset of the instruction is 0x1122.
  
```

- Disassemble the ELF file to check the instruction at `0x1122`.
```sh
  
$ objdump --disassemble libmath.so
0000000000001110 <divide>:
    1110:       f3 0f 1e fa             endbr64
    1114:       55                      push   %rbp
    1115:       48 89 e5                mov    %rsp,%rbp
    1118:       89 7d fc                mov    %edi,-0x4(%rbp)
    111b:       89 75 f8                mov    %esi,-0x8(%rbp)
    111e:       8b 45 fc                mov    -0x4(%rbp),%eax
    1121:       99                      cltd
    1122:       f7 7d f8                idivl  -0x8(%rbp)
    1125:       5d                      pop    %rbp
    1126:       c3                      ret
  
```

- **Observation**: The assembly code of the instruction at `0x1122` in the ELF file matches the instruction at `0x00007ffff7fb9122` in process memory. The assembly code of both are `idivl  -0x8(%rbp)`.


# References
- [Executable and Linking Format Files](https://man7.org/linux/man-pages/man5/elf.5.html)
- [Process Memory Map](https://man7.org/linux/man-pages/man5/proc_pid_maps.5.html)