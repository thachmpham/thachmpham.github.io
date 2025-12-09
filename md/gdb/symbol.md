---
title: "The Symbol Table"
---


# 1. Symbol Table
A symbol represents a function, a global variable, and other named entities. The symbol table is the section that maps these symbols to their corresponding addresses.

:::::::::::::: {.columns}
::: {.column width=25%}

**Creation**

The compiler generates the symbol table when building object files, typically storing it in the .symtab and .dynsym sections.

- Symbols in .symtab have addresses fixed at build time.
- Symbols in .dynsym are resolved at runtime.

:::
::: {.column width=25%}

**Symbol Resolution**

If an object file refers to a symbol defined in another, compiler search the symbol definition in other object files and libraries:

- If not found, it results in "undefined reference to" error.
- If found, it proceeds to the relocation step.

:::
::: {.column width=25%}

**Relocation**

The linker assigns memory addresses to the sections.

Address of symbols in .symtab is computed relative to the address of the section where it resides.

:::
::: {.column width=25%}

**Runtime**

The loader searches through the dynamic libraries to resolve the addresses of symbols in .dynsym.

The resolved addresses are then stored in the .got section.

:::
::::::::::::::

## 1.1. Struct Elf64_Sym
According to [man elf](https://man7.org/linux/man-pages/man5/elf.5.html), each entry in symbol table is represented by the Elf64_Sym or Elf32_Sym struct.

:::::::::::::: {.columns}
::: {.column width=50%}

```sh
$ pahole elf64_sym
```

```c
struct elf64_sym {          // offset size
  Elf64_Word      st_name;  // 0      4
  unsigned char   st_info;  // 4      1
  unsigned char   st_other; // 5      1
  Elf64_Half      st_shndx; // 6      2
  Elf64_Addr      st_value; // 8      8
  Elf64_Xword     st_size;  // 16     8
  /* size: 24 bytes */
};
```

:::
::: {.column width=50%}

<br>

<br>

| Field       |                                                                          |
|:------------|:-------------------------------------------------------------------------|
| `st_name`   | Symbol name                                                              |
| `st_info`   | Type, binding.                                                           |
| `st_other`  | Visibility.                                                              |
| `st_shndx`  | Section which the symbol resides                                         |
| `st_value`  | Offset to the start of section.                                          |
| `st_size`   | Size.                                                                    |

:::
::::::::::::::


## 1.2. Static Symbol Table
### 1.2.1. Section .symtab

:::::::::::::: {.columns}
::: {.column width=50%}

The .symtab section contains the static symbol table. The address of symbols in .symtab are resolved at building time.

Relationships to other sections.

- Field st_index represents the symbol name, which points to index of an entry in the string table .strtab.
- Field st_shndx represents the section that , which points to index of an entry in the section header table.

:::
::: {.column width=50%}

```go
┌────────────┐       ┌────────────┐        ┌────────────┐
│   .strtab  │       │  .symtab   │        │  section   |
┼────────────┤       ┼────────────┤        ┼────────────┤
│   index ◄──┼───────┼─ st_index  │   ┌────┼► index     │
│            │       │            │   │    │            │
│   value    │       │  st_shndx ─┼───┘    │  .....     │
└────────────┘       └────────────┘        └────────────┘
```

:::
::::::::::::::


### 1.2.2. Decode .symtab

:::::::::::::: {.columns}
::: {.column width=40%}

File main.c

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

:::
::: {.column width=60%}

List sections.

```sh
$ readelf --section-headers main
[Nr] Name       Type        Address          Off    Size   ES Flg Lk Inf Al
[14] .text      PROGBITS    0000000000001040 001040 00013b 00  AX  0   0 16
[23] .data      PROGBITS    0000000000004000 003000 000014 00  WA  0   0  8
[26] .symtab    SYMTAB      0000000000000000 003048 000378 18     27  18  8
[27] .strtab    STRTAB      0000000000000000 0033c0 0001d3 00      0   0  1
```

Extract section .symtab, which starts at offset 0x003048, size of 0x000378. The elf64_sym struct has a size of 24 bytes, so use option 'xxd -c 24' to display 24 bytes each line.

```sh
$ xxd -p -g 1 -c 24 -s 0x003048 -l 0x000378 main
340100001100170010400000000000000400000000000000
6301000012000e0029110000000000001800000000000000
8701000012000e0041110000000000003a00000000000000
```

:::
::::::::::::::

In x86_64, symbol entry is represented by struct elf64_sym. To decode struct elf64_sym from raw hex memory, we use script hex_to_elf64_sym.py.

```python
import struct
import sys

FMT_ELF64_SYM = "=IBBHQQ"

def hex_to_elf64_sym(hex_str: str):
    data = bytes.fromhex(hex_str)
    name, info, other, shndx, value, size = struct.unpack(FMT_ELF64_SYM, data)
    print("name info other shndx value size")
    print(hex(name), hex(info), hex(other),
            hex(shndx), hex(value), hex(size))

hex_to_elf64_sym(sys.argv[1])
```

:::::::::::::: {.columns}
::: {.column width=50%}

Decode the first line.

```sh
$ hex_to_elf64_sym.py 340100001100170010400000000000000400000000000000
  name     info    other  shndx   value     size
['0x134', '0x11', '0x0', '0x17', '0x4010', '0x4']
```

Decode the second line.

```sh
$ hex_to_elf64_sym.py 6301000012000e0029110000000000001800000000000000
  name     info    other  shndx  value     size
['0x163', '0x12', '0x0', '0xe', '0x1129', '0x18']
```

Print string table .strtab.

```sh
$ readelf --string-dump=.strtab main
[   134]  g_int
[   163]  sum
[   187]  main
```

:::
::: {.column width=50%}

Explain the outputs.

| First Line     |    |                                 |
|:-----------|:--------|:----------------------------------------|
| st_name    | 0x134   | Index 0x134 in .strtab: g_int   |
| st_info    | 0x11    | Type object, global          |
| st_other   | 0x0     | Visibility default                    |
| st_shndx   | 0x17    | Section 0x17 = 23 = .data              |
| st_value   | 0x4010  | Offset 0x4010                         |
| st_size    | 0x4     | Size of 4 bytes                           |


| Second Line      |    |                                 |
|:-----------|:--------|:----------------------------------------|
| st_name    | 0x163   | Index 0x136 in .strtab: sum |
| st_info    | 0x11    | Type func, global          |
| st_other   | 0x0     | Visibility default                     |
| st_shndx   | 0x17    | Section: 0xe = 14 = .text              |
| st_value   | 0x4010  | Offset 0x1129                         |
| st_size    | 0x18    | Size of 24 bytes |


:::
::::::::::::::

### 1.2.3. readelf .symtab
The goal of decoding .symtab is to understand the format of the symbol table. For everyday work, we can simply view .symtab using readelf.

:::::::::::::: {.columns}
::: {.column width=50%}

```sh
$ readelf --symbols main
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    23: 0000000000004010     4 OBJECT  GLOBAL DEFAULT   23 g_int
    27: 0000000000001129    24 FUNC    GLOBAL DEFAULT   14 sum
    32: 0000000000001141    58 FUNC    GLOBAL DEFAULT   14 main
```

:::
::: {.column width=50%}
Explain the output:

- The symbol g_int represents a 4-byte variable in the .data section. Within .data, the variable is positioned at offset 0x4010 from the start address of the section.
- The symbol sum represents a function in the .text section. Within .text, the function is positioned at offset 0x1129 from the start address of the section.

:::
::::::::::::::

## 1.3. Dynamic Symbol Table
### 1.3.1. Section .dynsym

:::::::::::::: {.columns}
::: {.column width=50%}

The .dynsym section contains the dynamic symbol table. The address of symbols this section are resolved at runtime by the loader, which search through the dynamic lirbaries to locate the symbols. 

Relationships to other sections.

- Field st_index represents the symbol name, which points to index of an entry in the string table .dynstr.

:::
::: {.column width=50%}

```go
┌────────────┐       ┌────────────┐
│   .dynstr  │       │  .dynsym   │
┼────────────┤       ┼────────────┤
│   index ◄──┼───────┼─ st_index  │
│            │       │            │
│   value    │       │  .....     |
└────────────┘       └────────────┘
```

:::
::::::::::::::


:::::::::::::: {.columns}
::: {.column width=60%}

Use readelf to print dynamic symbols in file.

```sh
$ readelf --wide --dyn-syms /usr/bin/ls
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    86: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND mempcpy@GLIBC_2.2.5 (3)
    87: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND memmove@GLIBC_2.2.5 (3)
   118: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND free@GLIBC_2.2.5 (3)
   120: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND malloc@GLIBC_2.2.5 (3)
```

:::
::: {.column width=40%}

<br>

Explain the output.

- Functions memcpy, memove, free, malloc are the functions defined in the external library libc.so, not in the program. So they appear in the dynamic symbol table for runtime linker to resolve at runtime.

:::
::::::::::::::


### 1.3.2. Sections .plt, .got

The .plt (procedure linkage table) section contains functions that helps program calls functions in dynamic libraries without knowing their actual memory addresses.  
The .got (global offset table) section contains the memory addresses of functions located in dynamic libraries.

:::::::::::::: {.columns}
::: {.column width=35%}

Interaction between .plt and .got.

**First call.**  
When program calls puts, the call goes to puts@plt. Because the symbol is not resolved yet, the plt redirects to runtime linker to:

- Look up the address of puts in libc.so.
- Call puts once.
- Write the resolved address to .got.

**Later calls.**  
Function puts@put simply read the resolved address from .got and jump directly to the real puts function.

:::
::: {.column width=65%}

```go
                                            ┌──────────────────┐                                  
┌──────────┐    ┌──────────────────────┐    │   ┌───────────┐  │   ┌─────────────────────────────┐
│ main()   │    │.plt                  │    │   │.got       │  │   │ runtime linker              │
│          │    |──────────────────────|    │   |───────────|  │   |─────────────────────────────|
│   puts()─┼────┼► puts@plt()          │    │   │           │  └───┼─►_dl_runtime_resolve()      │
│          │    │    find puts in .got─┼────┼──►│           │      │    find puts                │
└──────────┘    │    if not found      │    │   │           │◄─────┼─── update puts addr to .got │
                │       let ld.so find─┼────┘   └───────────┘      └─────────────────────────────┘
                │    call puts         │                                                          
                └──────────────────────┘                                                          
```

:::
::::::::::::::

:::::::::::::: {.columns}
::: {.column width=40%}

Sample: Examine the .plt and .got.

```c
#include <stdio.h>

int main(int argc, char** argv)
{
    puts("hello");    
    return 0;
}
```

Use '-fcf-protection=none' to place puts@plt to .plt instead of .plt.sec.

```sh
$ gcc -fcf-protection=none main.c -o main
```

Trace the call flow: puts@plt() to puts().

```sh
(gdb) start
(gdb) break puts@plt
Breakpoint 2 at 0x555555555030
(gdb) break puts
Breakpoint 3 at 0x7ffff7e08e50
(gdb) continue
Breakpoint 2, 0x0000555555555030 in puts@plt ()
(gdb) continue
Breakpoint 3, __GI__IO_puts (str=0x555555556004 "hello")
```

:::
::: {.column width=60%}

Check address of puts@plt(). It is function resides in .plt section.
```sh
(gdb) info address puts@plt
Symbol "puts@plt" is at 0x555555555030 in a file compiled without debugging.

(gdb) info symbol 0x555555555030
puts@plt in section .plt of /root/demo/main
```

Disassemble puts@plt(). It jumps to address 0x555555557fd0 that is an entry in .got.
```sh
(gdb) disassemble 0x555555555030
   0x0000555555555030 <+0>:     jmp    *0x2f9a(%rip)    # 0x555555557fd0 <puts@got.plt>
   0x0000555555555036 <+6>:     push   $0x0
   0x000055555555503b <+11>:    jmp    0x555555555020

(gdb) info symbol 0x555555557fd0
puts@got[plt] in section .got of /root/demo/main
```

Examine the .got entry. It stores address of function puts() in libc.so.
```sh
(gdb) x/a 0x555555557fd0
0x555555557fd0 <puts@got.plt>:  0x7ffff7e08e50 <__GI__IO_puts>

(gdb) info symbol 0x7ffff7e08e50
puts in section .text of /lib/x86_64-linux-gnu/libc.so.6
```

:::
::::::::::::::

Illustrate the process memory mappings.
```go
                 file: main                                    file: main                              file: libc.so
              section: .plt                                 section: .got                           section: .text
             function: puts@plt()                                                                   function: puts

┌────────────────────┬─────────────────────┐      ┌────────────────┬────────────────┐      ┌────────────────┬────────────────┐
│ address            │ value               │      │ address        │     value      │      │ address        │     value      │
┼────────────────────┼─────────────────────┤      ┼────────────────┼────────────────┤      ┼────────────────┼────────────────┤
│ 0x0000555555555030 │ jump  0x555555557fd0┼───┐  │                │                │  ┌───┼►0x7ffff7e08e50 │ endbr64        │
│                    │                     │   │  │                │                │  │   │                │                │
│ 0x0000555555555036 │ push  $0x0          │   └──┼►0x555555557fd0 │ 0x7ffff7e08e50─┼──┘   │ 0x7ffff7e08e54 │ push   %r14    │
│                    │                     │      │                │                │      │                │                │
│ 0x000055555555503b │ jmp   0x555555555020│      │                │                │      │ 0x7ffff7e08e56 │ push   %r13    │
└────────────────────┴─────────────────────┘      └────────────────┴────────────────┘      └────────────────┴────────────────┘
```

# 1. Usage
## 1.1. Data Type

:::::::::::::: {.columns}
::: {.column width=50%}

Name of data type.
```sh
  whatis expr
```

Definition of data type.
```sh
  ptype [/flags] expr
```

- *expr*: variable, struct, class name.
- *flag*: r, m, M, t, T, x, d, o (offset).


Find data type match a regex.
```sh
  info types [-q] [regex]
```

:::
::: {.column width=50%}

```cpp
class Point
{
public:
  int x, y;
  Point(int a, int b)  {}
  void move(int dx, int dy){}
};

int main() {
  Point p(1, 2);
  p.move(3, 4);
}

```

:::
::::::::::::::

Sample: Inspect a variable and a class.

```sh
(gdb) whatis p
type = Point
(gdb) ptype /o class Point
/* offset      |    size */  type = class Point {
                             public:
/*      0      |       4 */    int x;
/*      4      |       4 */    int y;

                               /* total size (bytes):    8 */
                             }
(gdb) info types -q Poi*
File demo.cpp:
1:      Point;

```

<br>

### 1.2. Function
Function names matches a regex.
```sh
  info functions [-q] [-n] [-t type_regex] [regex]
```

- *-q*: quite.
- *-n*: not print non-debug symbols.
- *-t*: only print functions whose type match type_regex. Check type by *whatis*.

:::::::::::::: {.columns}
::: {.column width=50%}

```c
int echo(int n)
{
  return 0;
}

char* echo(char* s)
{
  return 0;
}

```

:::
::: {.column width=50%}

```sh
(gdb) info function echo
File demo.cpp:
2:      char *echo(char*);
1:      int echo(int);

(gdb) info function -t char* echo
File demo.cpp:
2:      char *echo(char*)

(gdb) info function -t int echo
File demo.cpp:
1:      int echo(int);

```

:::
::::::::::::::


### 1.3. Variable
Global and static variables.
```sh
  info variables [-q] [-n] [-t type_regexp] [regexp]
```

<br>

### 1.4. Symbol to Address
Find address of a symbol.
```sh
  info address sym
```

:::::::::::::: {.columns}
::: {.column width=50%}

```c
int g = 0;

int sum(int a, int b)
{
    return a + b;
}

```

:::
::: {.column width=50%}

```sh
(gdb) info address g
Symbol "g" is static storage at address 0x420020.

(gdb) info address sum
Symbol "sum" is a function at address 0x40066c.

```

:::
::::::::::::::

<br>

### 1.5. Address to Symbol

:::::::::::::: {.columns}
::: {.column width=50%}

Find symbol at an address. Only global, static scope.
```sh
  info sym addr
```

:::
::: {.column width=50%}

```sh
(gdb) info symbol 0x420020
g in section .bss

(gdb) info symbol 0x40066c
sum in section .text

```

:::
::::::::::::::

<br>

### 1.6. Demangle

:::::::::::::: {.columns}
::: {.column width=50%}

Demangle a name.
```sh
  demangle [--] name
  set print demangle [on|off]

```

- `--`: useful when name begins with a dash.

:::
::: {.column width=50%}

```sh
(gdb) demangle _ZN5PointC2Eii
Point::Point(int, int)

(gdb) demangle _ZN5Point4moveEii
Point::move(int, int)

```

:::
::::::::::::::

<br>

### 1.7. Add Symbol
:::::::::::::: {.columns}
::: {.column width=40%}

GDB supports adding additional symbol files, it is especially useful when debugging code loaded by mmap.

To add the symbols.
```sh
  add-symbol-file filename
    [-readnow|-readnever]
    [-o offset]
    [textaddr]
    [-s section addr…]
```

To debug functions defined object file, we load the symbols for the .text section — the section that holds the executable code.

- *textaddr*: memory address where file’s text section loaded.

:::
::: {.column width=60%}

We can load the symbols for other sections as well, such as: .bss, .data to inspect the variables.

- *section*: section to load.
- *addr*: memory address where the section loaded.

Other options.

- *readnow*: load symbols immediately.
- *readnever*: not load symbol.
- *offset*: offset to add to the start address of each section, except those for which the address was specified explicitly.

<br>

To remove the symbols.
```sh
  remove-symbol-file filename
  remove-symbol-file -a addr

```

:::
::::::::::::::

<br>

<span style="color: yellow">Sample:</span>
Add symbols for memory regions loaded by mmap.

:::::::::::::: {.columns}
::: {.column width=35%}

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

<br>

Build.
```sh
$ gcc -c math.c -o math.o
$ gcc -g main.c -o main
```

:::
::: {.column width=65%}

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
::::::::::::::

Inspect the ELF file.
```sh
$ readelf --section-headers --wide math.o
[Nr] Name     Type        Address          Off    Size   ES Flg Lk Inf Al
[ 1] .text    PROGBITS    0000000000000000 000040 00002e 00  AX  0   0  1
[ 2] .data    PROGBITS    0000000000000000 000070 000008 00  WA  0   0  4

$ readelf --symbols math.o
Num:    Value          Size Type    Bind   Vis      Ndx Name
  3: 0000000000000000     4 OBJECT  GLOBAL DEFAULT    2 number1
  4: 0000000000000004     4 OBJECT  GLOBAL DEFAULT    2 number2
  5: 0000000000000000    24 FUNC    GLOBAL DEFAULT    1 sum
  6: 0000000000000018    22 FUNC    GLOBAL DEFAULT    1 sub

```

<span style="color: yellow">Compute ELF offsets.</span>

- 'readelf --section-headers' provides section offset from the start of the file.
- 'readelf --symbols' shows each symbol offset from the start of its section. Adding this offset to the section location gives the symbol location.

:::::::::::::: {.columns}
::: {.column width=45%}
From 'readelf --section-headers',

- Section .text at 0x40.
- Section .data at 0x70.

From 'readelf --symbols',

Symbol sum.

- In .text section (Ndx 1).
- Offset 0x00 in section .text (Value 0x00).

Symbol sub.

- In .text section (Ndx 1).
- Offset 0x18 in section .text (Value 0x18).

Symbol number1.

- In .data section (Ndx 2).
- Offset 0x00 in section .data (Value 0x00).
- Size of 4 bytes.

Symbol number2.

- In .data section (Ndx 2).
- Offset 0x04 in section .data (Value 0x04).
- Size of 4 bytes.

:::
::: {.column width=55%}

Sketch struct of the ELF file.

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

Summary:

|  |  |
|-----------------|---------|
| .text           | 0x40    |
| sum             | 0x40    |
| sub             | 0x58    |

|  |  |
|-----------------|---------|
| .data           | 0x70    |
| number1         | 0x70    |
| number2         | 0x74    |


:::
::::::::::::::


Start program.
```sh
(gdb) file main
(gdb) break main.c:19
(gdb) run
```

Print the memory mappings.
```sh
(gdb) info proc mappings
    Start Addr         End Addr   Size     Offset  Perms  objfile
0x7ffff7ffa000   0x7ffff7ffb000   0x1000   0x0     r-xp   math.o

```

<span style="color: yellow">Compute memory addresses.</span>

The offsets of sections within the ELF file, and the offsets of symbols within their sections, remain the same in memory.

So, once the start address of the file in memory is known, we can calculate the addresses of sections and symbols by adding their respective ELF offsets.

:::::::::::::: {.columns}
::: {.column width=45%}

From 'info proc mappings',

- math.o loaded at 0x7ffff7ffa000.


By adding corresponding ELF offsets,

Section .text:

- .text at 0x7ffff7ffa040.
- sum at 0x7ffff7ffa040.
- sub at 0x7ffff7ffa058.

Section .data:

- .data at 0x7ffff7ffa070.
- number1 at 0x7ffff7ffa070.
- number2 at 0x7ffff7ffa074.

:::
::: {.column width=55%}

Sketch the memory layout.

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


<span style="color: yellow">Add symbol file</span>

After loaded symbol file, GDB uses the given sections and addresses to determine the memory addresses of the symbols contained in those sections.

```sh
(gdb) add-symbol-file math.o 0x7ffff7ffa040 -s .data 0x7ffff7ffa070

(gdb) info files
0x00007ffff7ffa040 - 0x00007ffff7ffa06e is .text in /root/demo/math.o
0x00007ffff7ffa070 - 0x00007ffff7ffa078 is .data in /root/demo/math.o

```

:::::::::::::: {.columns}
::: {.column width=50%}

Print address of symbols.

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

Call functions, print variables.

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


<br>

### 1.8. Line
Print source line info.
```sh
  info line locspec
```

- *locspec*: function, address, etc.

<br>

Sample: Find line info of function, offset and instruction address.

:::::::::::::: {.columns}
::: {.column width=45%}

```c {.numberLines}
int triple(int a)
{
    return 3 * a;
}

int main()
{

}

```

:::
::: {.column width=55%}

```sh
(gdb) info line triple
Line 2 of "demo.c" starts at address 0x40066c <triple>
   and ends at 0x400674 <triple+8>.
(gdb) info line *triple+8
Line 3 of "demo.c" starts at address 0x400674 <triple+8>
   and ends at 0x400684 <triple+24>.
(gdb) info line *0x400674
Line 3 of "demo.c" starts at address 0x400674 <triple+8>
   and ends at 0x400684 <triple+24>.

```

:::
::::::::::::::

<br>