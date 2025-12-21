---
title: "Static & Dynamic Symbol Tables"
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

- Section header: provides list of sections in the file. Each symbol resides in one section.
- .strtab (string table): provides symbol names for .symtab.

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
The .strtab section is a string table that contains null-terminated strings. Each string represents a symbol name.

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
External symbol is a symbol whose address cannot be determined at compile time, but must be resolved at runtime.

.dynsym is the section that contains the list of external symbols. The .dynsym section works together with the .dynstr, .plt, .rela.plt, .got sections.

- .dynstr (dynamic string): provides symbol names for .dynsym.
- .plt (procedure linkage table): contains small executable code that executes runtime linking.
- .rela.plt (plt relocation table): tells the plt code how to resolve external symbols addresses.
- .got (global offset table): holds memory addresses of external symbol addresses.

<br>
Will examine the static symbol table of the below program.

Examine the .plt and .got.

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

## Section .dynsym
The .dynsym section is a string table that contains null-terminated strings. Each string represents a external symbol name.

```sh
$ readelf --string-dump=.dynstr main
String dump of section '.dynstr':
  ...  
  [    22]  puts
  ...
```

## Section .rela.plt
rela.plt is a relocation table that tells the runtime linker how to find addresses for external symbols.

:::::::::::::: {.columns}
::: {.column width=60%}

```sh
$ readelf --relocs main
Relocation section '.rela.plt' at offset 0x600 contains 1 entry:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000003fd0  000300000007 R_X86_64_JUMP_SLO 0000000000000000 puts@GLIBC_2.2.5 + 0
```

:::
::: {.column width=40%}

Explain the output.

- R_X86_64_JUMP_SLOT: Writes the memory address of the external symbol to the associated slot in .got. The slot size is 8 bytes.

- Write the memory address of puts to .got at offset 0x3fd0.

:::
::::::::::::::


## Sections .plt and .got
The .plt section contains a small executable code to find addresses for external symbols. The .got section holds the external symbol addresses.

```go
                                            ┌──────────────────┐
┌──────────┐    ┌──────────────────────┐    │   ┌───────────┐  │   ┌─────────────────────────────┐    ┌──────────┐
│ main()   │    │.plt                  │    │   │.got       │  │   │ runtime linker              │    │ rela.plt │
│          │    |──────────────────────|    │   |───────────|  │   |─────────────────────────────|    │          │
│   puts()─┼────┼► puts@plt()          │    │   │           │  └───┼─►_dl_runtime_resolve()      │    │          |
│          │    │    find puts in .got─┼────┼──►│           │      │    find puts                │    │          │
│          │    │                      │    │   │           │      │    check rela.plt ──────────┼──► │          |
└──────────┘    │    if not found      │    │   │           │◄─────┼─── update puts addr to .got │    └──────────┘
                │       let linker find┼────┘   └───────────┘      └─────────────────────────────┘
                │    call puts         │
                └──────────────────────┘
```

First call.  

- When program calls puts, the call goes to puts@plt. Since the symbol is not resolved yet, .plt delegates runtime linker to find the address of puts in libc.so.
- After found the address, it call puts.
- Following the instructions in .rela.plt, the resolved address is written to the corresponding entry in .got.

Next calls.  

- puts@put simply read the address of puts in .got and directly jump to the function.

<br>

Setup gdb to debug shared library events.
```sh
(gdb) set stop-on-solib-events 1
(gdb) set auto-solib-add 0
```

Start program.
```sh
(gdb) set environment LD_LIBRARY_PATH .
(gdb) file main
(gdb) start
```

### Before First Call
In the main function, when we call puts, it goes to puts@plt.
```sh
(gdb) disassemble main
Dump of assembler code for function main:
   0x0000555555555139 <+0>:     push   %rbp
   0x000055555555513a <+1>:     mov    %rsp,%rbp
   0x000055555555513d <+4>:     sub    $0x10,%rsp
   0x0000555555555141 <+8>:     mov    %edi,-0x4(%rbp)
   0x0000555555555144 <+11>:    mov    %rsi,-0x10(%rbp)
   0x0000555555555148 <+15>:    lea    0xeb5(%rip),%rax        # 0x555555556004
   0x000055555555514f <+22>:    mov    %rax,%rdi
   0x0000555555555152 <+25>:    call   0x555555555030 <puts@plt>
   0x0000555555555157 <+30>:    mov    $0x0,%eax
   0x000055555555515c <+35>:    leave  
   0x000055555555515d <+36>:    ret
```

puts@plt jumps to 0x555555557fd0 that is the address of the associated entry in .got.
```sh
(gdb) disassemble 0x555555555030
Dump of assembler code for function puts@plt:
   0x0000555555555030 <+0>:     jmp    *0x2f9a(%rip)        # 0x555555557fd0 <puts@got.plt>
   0x0000555555555036 <+6>:     push   $0x0
   0x000055555555503b <+11>:    jmp    0x555555555020

(gdb) info symbol 0x555555557fd0
puts@got[plt] in section .got of /root/demo/main
```

Since the address of puts is not yet resolved, the corresponding .got entry points back to 0x1036, an instruction in .plt, as shown in the objdump output. This indicates that the symbol has not been resolved, so the runtime linker is delegated to find the address.
```sh
(gdb) x/a 0x555555557fd0
0x555555557fd0 <puts@got.plt>:  0x1036
```

```sh
$ objdump --disassemble --section=.plt main
0000000000001030 <puts@plt>:
    1030:       ff 25 9a 2f 00 00       jmp    *0x2f9a(%rip)        # 3fd0 <puts@GLIBC_2.2.5>
    1036:       68 00 00 00 00          push   $0x0
    103b:       e9 e0 ff ff ff          jmp    1020 <_init+0x20>
```

Illustrate the call flow.
```go
                 file: main                                    file: main              
              section: .plt                                 section: .got            
             function: puts@plt()                                                    

┌────────────────────┬─────────────────────┐      ┌────────────────┬────────────────┐
│ address            │ value               │      │ address        │     value      │
┼────────────────────┼─────────────────────┤      ┼────────────────┼────────────────┤ 
│ 0x0000555555555030 │ jump  0x555555557fd0┼───┐  │                │                │ 
│                    │                     │   │  │                │                │
│ 0x0000555555555036 │ push  $0x0  ◄───────┼─┐ └──┼►0x555555557fd0 │ 0x1036 ──┐     │
│                    │                     │ │    │                │          │     │
│ 0x000055555555503b │ jmp   0x555555555020│ └────┼────────────────┼──────────┘     │
└────────────────────┴─────────────────────┘      └────────────────┴────────────────┘ 
```

Examine .got.
```sh
(gdb) info files
0x0000555555557fb8 - 0x0000555555558000 is .got

(gdb) x/9a 0x0000555555557fb8
0x555555557fb8: 0x3dc8  0x0
0x555555557fc8: 0x0     0x1036
0x555555557fd8: 0x0     0x0
0x555555557fe8: 0x0     0x0
0x555555557ff8: 0x0
```


### After First Call
Continue program, it will stop when libc.so loaded.
```sh
(gdb) continue
Stopped due to shared library event:
  Inferior loaded /lib/x86_64-linux-gnu/libc.so.6
```

Examine .got. After runtime loader solve puts, it updated the .got entry from 0x1036 to 0x7ffff7e08e50, which is the memory address of puts in libc.so.
```sh
(gdb) x/9a 0x0000555555557fb8
0x555555557fb8: 0x3dc8  0x0
0x555555557fc8: 0x0     0x7ffff7e08e50 <__GI__IO_puts>
0x555555557fd8: 0x7ffff7db1dc0 <__libc_start_main_impl> 0x0
0x555555557fe8: 0x0     0x0
0x555555557ff8: 0x7ffff7dcd9a0 <__cxa_finalize>

(gdb) info symbol 0x7ffff7e08e50
puts in section .text of /lib/x86_64-linux-gnu/libc.so.6
```

Check the call flow of puts@plt. Now, it jumps directly to 0x7ffff7e08e50, the memory address of puts in libc.so.
```sh
(gdb) disassemble 0x555555555030
Dump of assembler code for function puts@plt:
   0x0000555555555030 <+0>:     jmp    *0x2f9a(%rip)        # 0x555555557fd0 <puts@got.plt>
   0x0000555555555036 <+6>:     push   $0x0
   0x000055555555503b <+11>:    jmp    0x555555555020

(gdb) x/a 0x555555557fd0
0x555555557fd0 <puts@got.plt>:  0x7ffff7e08e50 <__GI__IO_puts>
```

Illustrate the call flow.
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

# Decode Symbol Table
## Struct Elf64_Sym

:::::::::::::: {.columns}
::: {.column width=50%}

The entry in symbol table is represented by the Elf64_Sym struct in x86_64 or Elf32_Sym struct in x86.

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

Relationships to other sections.

- Field st_index represents the symbol name, which points to index of an entry in the string table .strtab.
- Field st_shndx represents the section that , which points to index of an entry in the section header table.

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

To decode struct elf64_sym from raw hex memory, we use python script hex_to_elf64_sym.py.

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

## Decode .symtab

We will extract and decode .symtab of the below program.

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

Print string table .strtab.

```sh
$ readelf --string-dump=.strtab main
[   134]  g_int
[   163]  sum
[   187]  main
```

:::
::::::::::::::


:::::::::::::: {.columns}
::: {.column width=50%}

Extract section .symtab, which starts at offset 0x003048, size of 0x000378. The elf64_sym struct has a size of 24 bytes, so use option 'xxd -c 24' to display 24 bytes each line.

```sh
$ xxd -p -g 1 -c 24 -s 0x003048 -l 0x000378 main
340100001100170010400000000000000400000000000000
6301000012000e0029110000000000001800000000000000
8701000012000e0041110000000000003a00000000000000
```

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

To remove symbols.
```sh
  remove-symbol-file filename
  remove-symbol-file -a addr
```

:::
::: {.column width=50%}

Parameters:

- *filename*: the file contains the symbol table.

For .text section.

- *textaddr*: the memory address where .text of the file is loaded. It is required for debugging functions within .text.

For other sections, such as: .bss, .data,...

- *section*: section name.
- *addr*: memory address where the section is loaded.

Other options.

- *readnow*: load symbols immediately.
- *readnever*: not load symbol.
- *offset*: offset to add to the start address of each section, except sections specified addresses explicitly.

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


# References
- [ELF Manual](https://man7.org/linux/man-pages/man5/elf.5.html)