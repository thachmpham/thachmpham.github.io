---
title: "GDB: Memory"
---

## Find Memory
:::::::::::::: {.columns}
::: {.column width=60%}
Search memory for the sequence of bytes.
```sh
  find [/sn] start_addr, +len, vals...
  find [/sn] start_addr, end_addr, vals...
```

- *+len*: number of bytes to search.
- *s*: size of each search query value (b, h, w, g).
- *n*: number of matches to print.

:::
::: {.column width=40%}

```c
int main() 
{
  char str[] = "hello-hello";
  short a = 0x0102;
  int b = 0x0a0b0c0d;
}
```

:::
::::::::::::::

```python
# find str
(gdb) find &str, +sizeof(str), "hello"
(gdb) find &str, +sizeof(str), 'h', 'e', 'l', 'l', 'o'

# find a
(gdb) find &a, +2, (short)0x0102
(gdb) find /h &a, +2, 0x0102
(gdb) find /b &a, +2, 0x02, 0x01

# find b
(gdb) find &b, +4, (int)0x0a0b0c0d
(gdb) find /w &b, +4, 0x0a0b0c0d

# find b, a
#                  (int)b         , padding      , (short)a
(gdb) find &b, +8, (int)0x0a0b0c0d, (short)0x0000, (short)0x0102
```

<br>

## Export Memory
Dump the contents of memory from start_addr to end_addr, or the value of expr, to file.
```sh
  dump [format] memory filename start_addr end_addr
  dump [format] value filename expr
```
- *format*: binary, ihex, srec, tekhex, verilog.

```python
(gdb) p &str
$3 = (char (*)[12]) 0xffffffffec88

(gdb) dump memory mem.hex 0xffffffffec88 0xffffffffec88+12
(gdb) dump value val.hex str


$ hexdump -C mem.hex
00000000  68 65 6c 6c 6f 2d 68 65  6c 6c 6f 00              |hello-hello.|
$ hexdump -C val.hex 
00000000  68 65 6c 6c 6f 2d 68 65  6c 6c 6f 00              |hello-hello.|
```

<br>

## Import Memory
Restore the contents of file filename into memory.
```sh
  restore file [binary] offset start end
```

- *binary*: when file format is raw binary.
- *offset*: starting offset in file, default 0.
- *start*: starting address in target memory.
- *end*: ending address in target memory.

```python
(gdb) p &str
$3 = (char (*)[12]) 0xffffffffec88

#             file           start
(gdb) restore mem.hex binary 0xffffffffec88
Restoring binary file mem.hex into memory (0xffffffffec88 to 0xffffffffec94)

#             file           offset     start
(gdb) restore mem.hex binary 0          0xffffffffec88
Start address is greater than length of binary file mem.hex.

```

<br>

## Memory Regions
List memory regions.
```sh
  info proc mapping
```

<br>

## Core Dump
Generate the core dump.
```sh
  gcore [file]
```

- file: if file not specified, the file name defaults to core.pid, where pid is the inferior process ID.

Load the core dump.
```sh
  gdb program core
```