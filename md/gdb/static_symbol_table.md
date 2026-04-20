---
title: "Decode Static Symbol Table"
---


# Give a Try with readelf
## List of Sections

:::::::::::::: {.columns}
::: {.column}

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
::: {.column}

```sh
$ readelf --wide --sections main
Section Headers:
  [Nr] Name         Type        Address          Off    Size   ES Flg Lk Inf Al
  [ 6] .dynsym      DYNSYM      00000000000003d8 0003d8 000090 18   A  7   1  8
  [ 7] .dynstr      STRTAB      0000000000000468 000468 000088 00   A  0   0  1
  [12] .plt         PROGBITS    0000000000001020 001020 000010 10  AX  0   0 16
  [13] .plt.got     PROGBITS    0000000000001030 001030 000010 10  AX  0   0 16
  [14] .text        PROGBITS    0000000000001040 001040 00013b 00  AX  0   0 16
  [22] .got         PROGBITS    0000000000003fc0 002fc0 000040 08  WA  0   0  8
  [23] .data        PROGBITS    0000000000004000 003000 000014 00  WA  0   0  8
  [24] .bss         NOBITS      0000000000004014 003014 000004 00  WA  0   0  1
  [26] .symtab      SYMTAB      0000000000000000 003048 000378 18     27  18  8
  [27] .strtab      STRTAB      0000000000000000 0033c0 0001d3 00      0   0  1
  [28] .shstrtab    STRTAB      0000000000000000 003593 00010c 00      0   0  1
```

:::
::::::::::::::

- Index of .text section is 14 (Nr). It contains the executable machine code (PROBITS). In the file, it starts at offset 0x001040 and has a size of 0x00013b bytes.
- Index of .symtab section is 26 (Nr). It is a symbol table (SYMTAB). Start at 0x003048, length 0x000378 bytes.
- Index of .strtab section is 27 (Nr). It is a string table (STRTAB). Start at 0x0033c0, length 0x0001d3 bytes.

## Section .strtab
The .strtab section is a string table that contains null-terminated strings. Each string represents a symbol name.

:::::::::::::: {.columns}
::: {.column}

```sh
$ readelf --string-dump=.strtab main
String dump of section '.strtab':
  [   134]  g_int
  [   163]  sum
  [   187]  main  
```

:::
::: {.column}

Each line shows a symbol name and its offset within the .strtab section.

- String g_init starts at 0x134.
- String sum starts at 0x163.
- String main starts at 0x187.

:::
::::::::::::::

## Section .symtab
A symbol represents a function, a global variable, and other named entity. Section .symtab is the symbol table that map symbols to their addresses.

:::::::::::::: {.columns}
::: {.column width=50%}

```sh
$ readelf --wide --symbols main
Symbol table '.symtab' contains 37 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    23: 0000000000004010     4 OBJECT  GLOBAL DEFAULT   23 g_int
    27: 0000000000001129    24 FUNC    GLOBAL DEFAULT   14 sum
    32: 0000000000001141    58 FUNC    GLOBAL DEFAULT   14 main    
```

:::
::: {.column width=50%}

- Symbol g_int is a variable (OBJECT). In the .data section (Ndx=23), g_int starts at offset 0x4010 (Value), size of 4 bytes.
- Symbol sum is a function. In the .text section (Ndx=14), it starts at 0x1129, size of 4 bytes.
- Symbol main is a function. In the .text (Ndx=14), it starts at 0x1141, size of 58 bytes.

:::
::::::::::::::


# Decode Static Symbol Table

:::::::::::::: {.columns}
::: {.column}

Relationships between .symtab and other sections.

- .symtab.st_index represents a symbol name, points to an entry .strtab.
- .symtab.st_shndx represents a section, points to a section in header table.

:::
::: {.column}

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

## Struct Elf64_Sym

:::::::::::::: {.columns}
::: {.column width=40%}

In x86_64, a symbol is represented by struct Elf64_Sym. In x86_64, it is represented by Elf32_Sym.

```c
// pahole elf64_sym
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
::: {.column width=60%}

Script hex_to_elf64_sym.py decodes hex dump to struct elf64_sym.

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

:::
::::::::::::::


## Decode .symtab

:::::::::::::: {.columns}
::: {.column width=60%}

```sh
$ readelf --wide --sections main
[Nr] Name       Type        Address          Off    Size   ES Flg Lk Inf Al
[14] .text      PROGBITS    0000000000001040 001040 00013b 00  AX  0   0 16
[23] .data      PROGBITS    0000000000004000 003000 000014 00  WA  0   0  8
[26] .symtab    SYMTAB      0000000000000000 003048 000378 18     27  18  8
[27] .strtab    STRTAB      0000000000000000 0033c0 0001d3 00      0   0  1
```

:::
::: {.column width=40%}

```sh
$ readelf --string-dump=.strtab main
[   134]  g_int
[   163]  sum
[   187]  main
```

:::
::::::::::::::

1. Dump .symtab, which starts at 0x003048, size of 0x000378 bytes. Struct elf64_sym has a size of 24 bytes, so use 'xxd -c 24' to display 24 bytes per line.
```sh
$ xxd -p -g 1 -c 24 -s 0x003048 -l 0x000378 main
340100001100170010400000000000000400000000000000
6301000012000e0029110000000000001800000000000000
8701000012000e0041110000000000003a00000000000000
```

:::::::::::::: {.columns}
::: {.column width=60%}

2. Decode the first line in .symtab to elf64_sym.

```sh
$ hex_to_elf64_sym.py 340100001100170010400000000000000400000000000000
  name     info    other  shndx   value     size
['0x134', '0x11', '0x0', '0x17', '0x4010', '0x4']
```

3. Decode the second line in .symtab to elf64_sym.

```sh
$ hex_to_elf64_sym.py 6301000012000e0029110000000000001800000000000000
  name     info    other  shndx  value     size
['0x163', '0x12', '0x0', '0xe', '0x1129', '0x18']
```

:::
::: {.column width=40%}


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