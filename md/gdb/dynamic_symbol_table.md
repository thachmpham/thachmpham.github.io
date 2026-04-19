---
title: 'Dynamic Symbol Table'
---

# Read Dynamic Symbol Table
Dynamic symbol is a symbol whose address cannot be determined at compile time, but must be resolved at runtime. Section .dynsym contains the dynamic symbols. It works together with the .dynstr, .plt, .rela.plt, .got sections.

- .dynstr (dynamic string): provides symbol names for .dynsym.
- .plt (procedure linkage table): contains small executable code that executes runtime linking.
- .rela.plt (plt relocation table): tells the plt code how to resolve external symbols addresses.
- .got (global offset table): holds memory addresses of external symbol addresses.


## Section .dynstr

:::::::::::::: {.columns}
::: {.column width=50%}
Section .dynstr is a string table that contains null-terminated strings. Each string represents a external symbol name.

```sh
$ readelf --string-dump=.dynstr main
String dump of section '.dynstr':
  [    22]  puts
```

- String puts start at 0x22.

:::
::: {.column width=50%}

```c
#include <stdio.h>

int main(int argc, char** argv)
{
    puts("hello");    
    return 0;
}
```

```sh
# '-fcf-protection=none' to place puts@plt to .plt instead of .plt.sec.
$ gcc -fcf-protection=none main.c -o main
```

:::
::::::::::::::


## Section .rela.plt
rela.plt is a relocation table that tells the runtime linker how to find addresses for dynamic symbols.

```sh
$ readelf --relocs main
Relocation section '.rela.plt' at offset 0x600 contains 1 entry:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000003fd0  000300000007 R_X86_64_JUMP_SLO 0000000000000000 puts@GLIBC_2.2.5 + 0
```

- Type R_X86_64_JUMP_SLOT: Writes the memory address of the dynamic symbol to the associated slot in .got. The slot size is 8 bytes.
- Offset 000000003fd0: Write the memory address of puts to .got at 0x3fd0.


## Sections .plt and .got
The .plt section contains a small executable code to find addresses for dynamic symbols. The .got section holds the dynamic symbol addresses.

```go
                                            ┌──────────────────┐
┌──────────┐    ┌──────────────────────┐    │   ┌───────────┐  │   ┌─────────────────────────────┐
│ main()   │    │.plt                  │    │   │.got       │  │   │ runtime linker              │
│          │    |──────────────────────|    │   |───────────|  │   |─────────────────────────────|
│   puts()─┼────┼► puts@plt()          │    │   │           │  └───┼─►_dl_runtime_resolve()      │
│          │    │    find puts in .got─┼────┼──►│           │      │    find puts                │
│          │    │                      │    │   │           │      │                             │
└──────────┘    │    if not found      │    │   │           │◄─────┼─── update puts addr to .got │
                │       let linker find┼────┘   └───────────┘      └─────────────────────────────┘
                │    call puts         │
                └──────────────────────┘
```

- When program calls puts, the call goes to puts@plt. Since the symbol is not resolved yet, .plt delegates runtime linker to find the address of puts.
- After found the address, runtime linker update the address to .got and then execute the function.
- In the next calls, puts@plt simply read the address of puts in .got and directly jump to the function.

### Examine First Call

1. Setup gdb to debug shared library.
```sh
(gdb) set stop-on-solib-events 1
(gdb) set auto-solib-add 0

(gdb) set environment LD_LIBRARY_PATH .
(gdb) file main
(gdb) start
```

2. In the main function, when we call puts, it goes to puts@plt.
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

3. puts@plt jumps to 0x555555557fd0 that is the address of the associated entry in .got.
```sh
(gdb) disassemble 0x555555555030
Dump of assembler code for function puts@plt:
   0x0000555555555030 <+0>:     jmp    *0x2f9a(%rip)        # 0x555555557fd0 <puts@got.plt>
   0x0000555555555036 <+6>:     push   $0x0
   0x000055555555503b <+11>:    jmp    0x555555555020

(gdb) info symbol 0x555555557fd0
puts@got[plt] in section .got of /root/demo/main
```

4. Since the address of puts is not yet resolved, the corresponding .got entry points back to 0x1036, an instruction in .plt, as shown in the objdump output. This indicates that the symbol has not been resolved, so the runtime linker is delegated to find the address.
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

5. Examine .got.
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

Illustrate .plt, .got before the first call.
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


6. Continue program, it will stop when libc.so loaded.
```sh
(gdb) continue
Stopped due to shared library event:
  Inferior loaded /lib/x86_64-linux-gnu/libc.so.6
```

7. Examine .got. After runtime loader solve puts, it updated the .got entry from 0x1036 to 0x7ffff7e08e50, which is the memory address of puts in libc.so.
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

8. Check the call flow of puts@plt. Now, it jumps directly to 0x7ffff7e08e50, the memory address of puts in libc.so.
```sh
(gdb) disassemble 0x555555555030
Dump of assembler code for function puts@plt:
   0x0000555555555030 <+0>:     jmp    *0x2f9a(%rip)        # 0x555555557fd0 <puts@got.plt>
   0x0000555555555036 <+6>:     push   $0x0
   0x000055555555503b <+11>:    jmp    0x555555555020

(gdb) x/a 0x555555557fd0
0x555555557fd0 <puts@got.plt>:  0x7ffff7e08e50 <__GI__IO_puts>
```

Illustrate .plt, .got after the first call.
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