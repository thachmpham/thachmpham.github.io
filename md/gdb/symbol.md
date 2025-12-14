---
title: "Symbol Table"
---


# Symbol Table
A symbol represents a function, a global variable, and other named entity. The symbol table is a section that maps symbols to their addresses. Two typical symbol tables are the .symtab and .dynsym sections.

:::::::::::::: {.columns}
::: {.column width=50%}

The .symtab section.

- Contains all symbols in the file.
- Used by the static linker at build time to resolve symbol references between object files and perform relocations.
- Can be stripped from the binary without affecting program execution.

:::
::: {.column width=50%}

The .dynsym section.

- A subset of .symtab, contains only dynamic symbols.
- Used by dynamic linker at runtime to resolve external references to shared libraries.
- Cannot be stripped from the binary because it is required for program execution.

:::
::::::::::::::

<br>


# Static Symbol Table
The .symtab section is associated with the section header and the .strtab section.

- Each symbol resides in a section described by the section header.
- Each symbol name is taken from the .strtab section.

<br>

Will examine the static symbol table of the below program.

```c
int g_int = 0xcafebabe;

int sum(int x, int y)
{
    return x + y;
}

int main(int argc, char** argv)
{
    int a = 1, b = 2;
    int s = sum(a, b);
    return 0;
}
```

```sh
$ gcc main.c -o main
```


## Section Header
The section header provides a list of sections and their details in the ELF file.

:::::::::::::: {.columns}
::: {.column width=60%}

```sh
$ readelf --wide --sections main
Section Headers:
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  ...
  [ 6] .dynsym           DYNSYM          00000000000003d8 0003d8 000090 18   A  7   1  8
  [ 7] .dynstr           STRTAB          0000000000000468 000468 000088 00   A  0   0  1
  ...
  [12] .plt              PROGBITS        0000000000001020 001020 000010 10  AX  0   0 16
  [13] .plt.got          PROGBITS        0000000000001030 001030 000010 10  AX  0   0 16
  [14] .text             PROGBITS        0000000000001040 001040 00013b 00  AX  0   0 16
  ...
  [22] .got              PROGBITS        0000000000003fc0 002fc0 000040 08  WA  0   0  8
  [23] .data             PROGBITS        0000000000004000 003000 000014 00  WA  0   0  8
  [24] .bss              NOBITS          0000000000004014 003014 000004 00  WA  0   0  1
  ...
  [26] .symtab           SYMTAB          0000000000000000 003048 000378 18     27  18  8
  [27] .strtab           STRTAB          0000000000000000 0033c0 0001d3 00      0   0  1
  [28] .shstrtab         STRTAB          0000000000000000 003593 00010c 00      0   0  1
```

:::
::: {.column width=40%}

Explain the output.

- There are 28 sections.

- The .text section has index 14 (Nr). It contains the executable machine code (PROBITS). In the file, it starts at offset 0x001040 and has a size of 0x00013b bytes.

- The index of .symtab is 26 (Nr). It is a symbol table (SYMTAB). Within the file, it begins 0x003048 with a length of 0x000378 bytes.

- The .strtab section, with index 27 (Nr), is a string table (STRTAB). It starts at offset 0x0033c0 and spans 0x0001d3 bytes.

:::
::::::::::::::

## Section .strtab
The .strtab section contains a list of null-terminated strings. Each string represents a symbol name.

:::::::::::::: {.columns}
::: {.column width=60%}

```sh
$ readelf --string-dump=.strtab main
String dump of section '.strtab':
  ...
  [   134]  g_int
  ...
  [   163]  sum
  ...
  [   187]  main
  ...
  [   1cd]  _init
```

:::
::: {.column width=40%}

Explain the output.

- Each line shows a symbol name and its offset within the .strtab section.

- The string g_init is located at offset 0x134.

- The string sum is located at offset 0x163.

- The string main is located at offset 0x187.

:::
::::::::::::::


## Section .symtab
The .symtab section contains all symbols in the file.

:::::::::::::: {.columns}
::: {.column width=60%}

```sh
>>> readelf --wide --symbols main
Symbol table '.symtab' contains 37 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    ...
    23: 0000000000004010     4 OBJECT  GLOBAL DEFAULT   23 g_int
    ...
    27: 0000000000001129    24 FUNC    GLOBAL DEFAULT   14 sum
    ...
    32: 0000000000001141    58 FUNC    GLOBAL DEFAULT   14 main
    ...
    36: 0000000000001000     0 FUNC    GLOBAL HIDDEN    11 _init
```

:::
::: {.column width=40%}

Explain the output.

- The g_int symbol represents a variable (OBJECT), with size of 4 bytes (Size). In the .data section (Ndx=23, section index), g_int located at offset 0x4010 (Value).

- The sum symbol is a function (FUNC). Within the .text section (Ndx=14), sum located at 0x1129 (Value).

- The main symbol is a function (FUNC). In .text (Ndx=14), main located at 0x1141.

:::
::::::::::::::


# Dynamic Symbol Table
The .dynsym section works in conjunction with .dynstr, .plt, .


# GDB
## Add Symbols
:::::::::::::: {.columns}
::: {.column width=50%}

GDB supports adding symbol table from file, it is useful when debugging code loaded by mmap.

To add the symbols.
```sh
  add-symbol-file filename
    [-readnow|-readnever]
    [-o offset]
    [textaddr]
    [-s section addr…]
```

Parameters:

- *filename*: the file contains the symbol table.

Debug the .text section.

- *textaddr*: the memory address where .text of the file is loaded. It is required for debugging functions within .text.


:::
::: {.column width=50%}

We can also add symbols for other sections, such as: .bss, .data,... by the below parameters.

- *section*: section name.
- *addr*: memory address where the section is loaded.

Other options.

- *readnow*: load symbols immediately.
- *readnever*: not load symbol.
- *offset*: offset to add to the start address of each section, except those for which the address was specified explicitly.

<br>

To remove symbols.
```sh
  remove-symbol-file filename
  remove-symbol-file -a addr
```

:::
::::::::::::::

## Add Symbols to mmap Regions
In the below example, we will add symbols for memory regions that is loaded by mmap.

:::::::::::::: {.columns}
::: {.column width=60%}

File main.c
```c
#include <sys/mman.h>
#include <stddef.h>
#include <fcntl.h>

int main()
{
    int fd = open("math.o", O_RDONLY);

    // load math.o to memory
    mmap(
        NULL, // let kernel choose start addr of region
        4096, // length of the region
        PROT_READ|PROT_EXEC, // protection mode
        MAP_PRIVATE,         // visible mode
        fd,                  // file to load
        0                    // offset in file
    );

    return 0;
}

```

:::
::: {.column width=40%}

File math.c
```c
int number1 = 0xCAFEBABE;
int number2 = 0xC1A0C1A0;

int sum(int x, int y)
{
    return x + y;
}

int sub(int x, int y)
{
    return x - y;
}

```

Build.

```sh
$ gcc -c math.c -o math.o
$ gcc -g main.c -o main
```

:::
::::::::::::::


:::::::::::::: {.columns}
::: {.column width=60%}

Sections in math.o.
```sh
$ readelf --section-headers --wide math.o
[Nr] Name     Type        Address          Off    Size   ES Flg Lk Inf Al
[ 1] .text    PROGBITS    0000000000000000 000040 00002e 00  AX  0   0  1
[ 2] .data    PROGBITS    0000000000000000 000070 000008 00  WA  0   0  4
```

Symbols in math.o.
```sh
$ readelf --symbols math.o
Num:    Value          Size Type    Bind   Vis      Ndx Name
  3: 0000000000000000     4 OBJECT  GLOBAL DEFAULT    2 number1
  4: 0000000000000004     4 OBJECT  GLOBAL DEFAULT    2 number2
  5: 0000000000000000    24 FUNC    GLOBAL DEFAULT    1 sum
  6: 0000000000000018    22 FUNC    GLOBAL DEFAULT    1 sub
```

- Section .text locates at offset 0x40.
- Section .data locates at offset 0x70.
- Within .text, symbols sum, sub locates at offsets 0x00, 0x18.
- Within .data, symbols number1, number2 locates offsets 0x00, 0x14.

:::
::: {.column width=40%}

Illustrate math.o.

```go
+---------------------+
| ELF File Header     | 0x00
+---------------------+
| Program Headers     |
+---------------------+
| .text Section       | 0x40
| ├── sum()           |      + 0x00
| └── sub()           |      + 0x18
|                     |
+---------------------+
| .data Section       | 0x70
| ├── number1         |      + 0x00
| └── number2         |      + 0x04
|                     |
+---------------------+
| Other Sections...   |
+---------------------+
| Section Headers     |
+---------------------+

```

:::
::::::::::::::


:::::::::::::: {.columns}
::: {.column width=60%}


Start program.
```sh
(gdb) file main
(gdb) break main.c:19
(gdb) run
```

Print memory mappings. 
```sh
(gdb) info proc mappings
    Start Addr         End Addr   Size     Offset  Perms  objfile
0x7ffff7ffa000   0x7ffff7ffb000   0x1000   0x0     r-xp   math.o
```

When math.o is loaded into memory, the offsets of sections and symbols remain the same as in the ELF file. Once the memory address of the file is known, the addresses of sections and symbols can be calculated by adding their ELF offsets.

In memory, math.o is loaded at address 0x7ffff7ffa000. So, within math.o:

- .text locates at 0x7ffff7ffa040 (math.o base address + 0x40).
- .data locates at 0x7ffff7ffa070 (math.o base address + 0x70).

:::
::: {.column width=40%}

Illustrate memory.

```go
+---------------------+
| Other Regions...    |
+---------------------+
| math.o              | 0x7ffff7ffa000
|                     |
|   .text             | 0x7ffff7ffa040
|   ├── sum()         |         + 0x00
|   └── sub()         |         + 0x18
|                     |
|   .data             | 0x7ffff7ffa070
|   ├── number1       |         + 0x00
|   └── number2       |         + 0x40
+---------------------+
| Other Regions...    |
+---------------------+
```

:::
::::::::::::::

We need to add symbols in .text and .data of math.o, so we provide memory address of these sections to GDB.

```sh
(gdb) add-symbol-file math.o 0x7ffff7ffa040 -s .data 0x7ffff7ffa070
```

Check the loaded files and sections.
```sh
(gdb) info files
0x00007ffff7ffa040 - 0x00007ffff7ffa06e is .text in /root/demo/math.o
0x00007ffff7ffa070 - 0x00007ffff7ffa078 is .data in /root/demo/math.o
```

:::::::::::::: {.columns}
::: {.column width=50%}

Check the symbol addresses.

```sh
(gdb) info function sum
0x00007ffff7ffa040  sum

(gdb) info function sub
0x00007ffff7ffa058  sub

(gdb) info variable number1
0x00007ffff7ffa070  number1

(gdb) info variable number2
0x00007ffff7ffa074  number2
```

:::
::: {.column width=50%}

After loaded symbols, we can print and call them through symbol names.

```sh
(gdb) call (int) sum(4, 5)
$3 = 9

(gdb) call (int) sub(4, 5)
$5 = -1

(gdb) print/x (int)number1
$1 = 0xcafebabe

(gdb) print/x (int)number2
$2 = 0xc1a0c1a0
```

:::
::::::::::::::

