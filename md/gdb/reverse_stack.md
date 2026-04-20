---
title: 'Reverse the Stack Memory'
---

# Stack Pointer

Stack pointer (rsp) always points to top of the stack. Register rsp changed on instructions push, pop, call, ret.

## Push & Pop

Instruction push put a value onto the stack. Instruction pop remove a value from the stack.

:::::::::::::: {.columns}
::: {.column width=40%}

File main.s
```asm
.section .text
.global _start

.text
_start:
    movabs  $0xabcdabcdabcdabcd, %rax
    push    %rax

    movabs  $0x1234123412341234, %rax
    push    %rax

    movq    $60, %rax     # syscall: exit
    xorq    %rdi, %rdi    # exit code 0
    syscall
```

Build.
```sh
as main.s -o main.o
ld main.o -o main
```

Debug.
```sh
(gdb) file main
(gdb) break _start
(gdb) run
Breakpoint 1, 0x0000000000401000 in _start ()
```

:::
::: {.column width=60%}

Monitor stack.
```sh
(gdb) disassemble
Dump of assembler code for function _start:
=> 0x0000000000401000 <+0>:     movabs $0xabcdabcdabcdabcd,%rax
   0x000000000040100a <+10>:    push   %rax
   0x000000000040100b <+11>:    movabs $0x1234123412341234,%rax
   0x0000000000401015 <+21>:    push   %rax
   0x0000000000401016 <+22>:    mov    $0x3c,%rax
   0x000000000040101d <+29>:    xor    %rdi,%rdi
   0x0000000000401020 <+32>:    syscall

(gdb) while 1
> x/4gx $rsp
> x/i $rip
> nexti
> end
```

```sh
0x7fffffffe700: 0x0000000000000001      0x00007fffffffe9ab
0x7fffffffe710: 0x0000000000000000      0x00007fffffffe9bc

=> 0x401000 <_start>:   movabs $0xabcdabcdabcdabcd,%rax
=> 0x40100a <_start+10>:        push   %rax
0x7fffffffe6f8: 0xabcdabcdabcdabcd      0x0000000000000001
0x7fffffffe708: 0x00007fffffffe9ab      0x0000000000000000

=> 0x40100b <_start+11>:        movabs $0x1234123412341234,%rax
=> 0x401015 <_start+21>:        push   %rax
0x7fffffffe6f0: 0x1234123412341234      0xabcdabcdabcdabcd
0x7fffffffe700: 0x0000000000000001      0x00007fffffffe9ab
```

:::
::::::::::::::

Illustrate.
```go
┌───────────────┬───────────────────┐        ┌───────────────┬───────────────────┐        ┌───────────────┬───────────────────┐
│   address     │       value       │        │   address     │       value       │        │   address     │       value       │
├───────────────┼───────────────────┤        ├───────────────┼───────────────────┤        ├───────────────┼───────────────────┤
│               │                   │        │               │                   │        │               │                   │
│               │                   │ push   │               │                   │ push   │0x7fffffffe6f0 │ 0x1234123412341234│
│               │                   │ ─────► │0x7fffffffe6f8 │ 0xabcdabcdabcdabcd│ ─────► │0x7fffffffe6f8 │ 0xabcdabcdabcdabcd│
│0x7fffffffe700 │ 0x0000000000000001│        │0x7fffffffe700 │ 0x0000000000000001│        │0x7fffffffe700 │ 0x0000000000000001│
│0x7fffffffe708 │ 0x00007fffffffe9ab│        │0x7fffffffe708 │ 0x00007fffffffe9ab│        │0x7fffffffe708 │ 0x00007fffffffe9ab│
│0x7fffffffe710 │ 0x0000000000000000│        │0x7fffffffe710 │ 0x0000000000000000│        │0x7fffffffe710 │ 0x0000000000000000│
│0x7fffffffe718 │ 0x00007fffffffe9bc│        │0x7fffffffe718 │ 0x00007fffffffe9bc│        │0x7fffffffe718 │ 0x00007fffffffe9bc│
└───────────────┴───────────────────┘        └───────────────┴───────────────────┘        └───────────────┴───────────────────┘
```


## Call & Ret

Instruction call pushes the address of the next instruction (rip) onto the stack then jump to the target function. This address is also called the return address. Instruction ret pops the return address from the stack and jump to this address.

:::::::::::::: {.columns}
::: {.column width=40%}

File main.s
```asm
.section .text
.global _start

_start:    
    mov     $5, %rdi    # x = 5
    mov     $7, %rsi    # y = 7
    call    sum         # sum(x, y)

    mov     $60, %rax   # syscall: exit
    xor     %rdi, %rdi  # exit code 0
    syscall

# add two integers
sum:
    add     %rsi, %rdi  # x += y
    mov     %rdi, %rax  # rax = x
    ret
```

```sh
as main.s -o main.o
ld main.o -o main
```

```sh
(gdb) file main
(gdb) break _start
(gdb) run
Breakpoint 1, 0x0000000000401000 in _start ()
```

:::
::: {.column width=60%}

Monitor stack.
```sh
(gdb) disassemble
Dump of assembler code for function _start:
=> 0x0000000000401000 <+0>:     mov    $0x5,%rdi
   0x0000000000401007 <+7>:     mov    $0x7,%rsi
   0x000000000040100e <+14>:    call   0x40101f <sum>
   0x0000000000401013 <+19>:    mov    $0x3c,%rax
   0x000000000040101a <+26>:    xor    %rdi,%rdi
   0x000000000040101d <+29>:    syscall

(gdb) while 1
> x/4gx $rsp
> x/i $rip
> stepi
> end
```

```sh
0x7fffffffe700: 0x0000000000000001      0x00007fffffffe9ab
0x7fffffffe710: 0x0000000000000000      0x00007fffffffe9bc

=> 0x40100e <_start+14>:        call   0x40101f <sum>
0x7fffffffe6f8: 0x0000000000401013      0x0000000000000001
0x7fffffffe708: 0x00007fffffffe9ab      0x0000000000000000

=> 0x401025 <sum+6>:    ret    
0x7fffffffe700: 0x0000000000000001      0x00007fffffffe9ab
0x7fffffffe710: 0x0000000000000000      0x00007fffffffe9bc
```

:::
::::::::::::::

Illustrate.
```go
┌───────────────┬───────────────────┐        ┌───────────────┬───────────────────┐        ┌───────────────┬───────────────────┐
│   address     │       value       │        │   address     │       value       │        │   address     │       value       │
├───────────────┼───────────────────┤        ├───────────────┼───────────────────┤        ├───────────────┼───────────────────┤
│               │                   │        │               │                   │        │               │                   │
│               │                   │ call   │               │                   │ ret    │               │                   │
│               │                   │ ─────► │0x7fffffffe6f8 │ 0x0000000000401013│ ─────► │               │                   │
│0x7fffffffe700 │ 0x0000000000000001│        │0x7fffffffe700 │ 0x0000000000000001│        │0x7fffffffe700 │ 0x0000000000000001│
│0x7fffffffe708 │ 0x00007fffffffe9ab│        │0x7fffffffe708 │ 0x00007fffffffe9ab│        │0x7fffffffe708 │ 0x00007fffffffe9ab│
│0x7fffffffe710 │ 0x0000000000000000│        │0x7fffffffe710 │ 0x0000000000000000│        │0x7fffffffe710 │ 0x0000000000000000│
│0x7fffffffe718 │ 0x00007fffffffe9bc│        │0x7fffffffe718 │ 0x00007fffffffe9bc│        │0x7fffffffe718 │ 0x00007fffffffe9bc│
└───────────────┴───────────────────┘        └───────────────┴───────────────────┘        └───────────────┴───────────────────┘
```

- Instruction call pushes $rip 0x0000000000401013 to the stack, and jump to the sum function.
- Instruction ret pops 0x0000000000401013 from the stack, and then jump to this address.


# Base Pointer

Base pointer (rbp) stores the start address of the function stack frame. Remains fixed throughout the function, allowing compiler to use its stable position to generate offsets for local variables and function arguments.

## Prolog & Epilog

:::::::::::::: {.columns}
::: {.column width=40%}

When calling a new C/C++ function, prolog is the compiler-generated code that sets up the stack frame for the new function. When leaving a C/C++ function, epilog is the compiler-generated code that restores the parent's stack frame and jump back to parent function.

Example: parent_func calls child_func.
```c
parent_func() {
    child_func()
}

child_func() {

}
```

:::
::: {.column width=60%}

```asm
<parent_func>:
call    child_func  # save parent next instruction address (rip) to stack,
                    # jump to child_func.
```

```asm
<child_func>:
                    # prolog
push   %rbp         # save parent base pointer (rbp) to stack.
mov    %rsp, %rbp   # move current base pointer to top of stack (rsp).

...                 # function body

                    # epilog
pop    %rbp         # restore parent base pointer from stack.
ret                 # restore parent next instruction address from stack,
                    # jump to this instruction (parent_func).
```

:::
::::::::::::::

Illustrate.
```go
        ┌───────────┐              ┌───────────┐             ┌───────────┐             ┌───────────┐
        │   stack   │              │   stack   │             │   stack   │             │   stack   │
$rbp ─┐ ├───────────┤              ├───────────┤    $rbp ─┐  ├───────────┤    $rbp ──┐ ├───────────┤
      │ │           │              │           │          │  │           │           │ │           │
      │ │           │              │           │          │  │           │           │ │           │
      │ │           │              │           │          │  │           │           │ │           │
      │ │           │  parent_func │           │          └─►│parent $rbp│           │ │           │
      │ │           │  call        │parent $rip│             │parent $rip│           │ │           │
      └─┼►          │  child_func  │           │   prolog    │           │  epilog   └─┼►          │
        │           │ ───────────► │           │ ──────────► │           │ ──────────► │           │
        └───────────┘              └───────────┘             └───────────┘             └───────────┘
```

## Arguments & Local Variables

Local variables locates on the stack for each function call. Their addresses are based on negative offsets from the base pointer rbp.

:::::::::::::: {.columns}
::: {.column width=50%}

```c
int a, b, c;
a = 1;
b = 2;
c = 3;
```

:::
::: {.column width=50%}

```asm

movl   $0x1,-0xc(%rbp)  ; a at $rbp-0xc
movl   $0x2,-0x8(%rbp)  ; b at $rbp-0x8
movl   $0x3,-0x4(%rbp)  ; c at $rbp-0x4
```

:::
::::::::::::::

When the parent function calls the child function, the arguments are saved in the registers rdi, rsi, rdx, rcx, r8, and r9.
If more than 6 arguments, the extra arguments are saved on the stack.  
Inside the child function, it reads the registers and saves the arguments into its stack frame. The argument addresses in the stack frame are based on the base pointer rbp.

:::::::::::::: {.columns}
::: {.column width=50%}

```c
parent_function()
{
    child_func(a, b);
}

int child_func(int a, int b)
{
}
```

:::
::: {.column width=50%}

```asm
<parent_function>:
  mov    -0x8(%rbp),%esi    # save a to $esi
  mov    -0xc(%rbp),%edi    # save b to $edi
  call   child_func

<child_func>:
  mov    %edi,-0x14(%rbp)   # save $edi to stack, b
  mov    %esi,-0x18(%rbp)   # save $esi to stack, a
```

:::
::::::::::::::

# Print the Raw Stack

:::::::::::::: {.columns}
::: {.column width=50%}

Instruction call pushes the parent rip onto the stack. Prolog pushes the parent rip onto the stack. Function arguments and local variables are stored in the stack, and their addresses are based on the current $rbp.

So, from current rbp, go backward to lower addresses, we reach the local variables and arguments. From current rbp, go forward to higher addresses, we reach the parent rbp and rip.

In GDB:

- To examine arguments and local variables, examine backward from rbp, eg: `x/-6wx $rbp`.
- To examine the parent’s rbp and rip, examine forward from rbp: `x/2a $rbp`.

:::
::: {.column width=50%}

```go
───low address  ┌────────────────┐
                │     stack      │
                ├────────────────┤
                │                │
                │ ...            │
                │ argument       │
                │ argument       │
                │ ...            │
                │ local variable │
                │ local variable │
      $rbp ───► │ parent $rbp    │
                │ parent %rip    │
                │                │
──high address  └────────────────┘
```

:::
::::::::::::::

:::::::::::::: {.columns}
::: {.column width=50%}

1. main.c
```c
int func(int argv1, int argv2)
{
    int a = 0xaaaaaaaa;
    int b = 0xbbbbbbbb;
    int c = 0xcccccccc;
    int d = 0xdddddddd;
    return 0;
}

int main(int argc, char** argv)
{
    func(0x11111111, 0x22222222);
    return 0;
}
```

```sh
$ gcc -g main.c -o main
```

2. Run.

```sh
(gdb) file main
(gdb) disassemble func
   0x0000000000001129 <+0>:     endbr64 
   0x000000000000112d <+4>:     push   %rbp
   0x000000000000112e <+5>:     mov    %rsp,%rbp
   0x0000000000001131 <+8>:     mov    %edi,-0x14(%rbp)
   0x0000000000001134 <+11>:    mov    %esi,-0x18(%rbp)
   0x0000000000001137 <+14>:    movl   $0xaaaaaaaa,-0x10(%rbp)
   0x000000000000113e <+21>:    movl   $0xbbbbbbbb,-0xc(%rbp)
   0x0000000000001145 <+28>:    movl   $0xcccccccc,-0x8(%rbp)
   0x000000000000114c <+35>:    movl   $0xdddddddd,-0x4(%rbp)
   0x0000000000001153 <+42>:    mov    $0x0,%eax
   0x0000000000001158 <+47>:    pop    %rbp
   0x0000000000001159 <+48>:    ret
```

:::
::: {.column width=50%}

3. Set breakpoint at the end of func.

```sh
(gdb) break *func+42

(gdb) run
Breakpoint 1, 0x0000555555555153 in func ()
```

4. Examine arguments and local variables.
```sh
(gdb) x/-6wx $rbp
0x7fffffffe5b8: 0x22222222      0x11111111      0xaaaaaaaa      0xbbbbbbbb
0x7fffffffe5c8: 0xcccccccc      0xdddddddd
```

5. Examine parent's $rip and $rbp.
```sh
(gdb) x/2a $rbp
0x7fffffffe5d0: 0x7fffffffe5f0  0x55555555517c <main+34>
```

Illustrate the stack.
```go
                         ┌────────────────┐
                         │     stack      │
          ───low address ├────────────────┤
                         │                │
          0x7fffffffe5b8 │ 0x22222222     │ argument argv2
                         │ 0x11111111     │ argument argv2
                         │ 0xaaaaaaaa     │ variable a
                         │ 0xbbbbbbbb     │ variable b
          0x7fffffffe5c8 │ 0xcccccccc     │ variable c
                         │ 0xdddddddd     │ variable d
$rbp ───► 0x7fffffffe5d0 │ 0x7fffffffe5f0 │ parent $rbp
                         │ 0x55555555517c │ parent $rip
                         │                │
          ──high address └────────────────┘
```

:::
::::::::::::::

# Reverse Function Arguments
## DWARF to Argument Addresses

Fomular to calculate function argument and local variable addresses.
```sh

    Address = DW_AT_location + CFA_Offset


    While:
        DW_AT_location is the relative address of the argument. Can be in section .debug_info.
        CFA_Offset is Call Frame Address Offset at target instruction. Can be found in section .eh_frame.

```

<br>

1. Find range of func().
```sh
$ readelf --debug-dump main
```

```sh
Contents of the .debug_info section:

 <1><85>: Abbrev Number: 8 (DW_TAG_subprogram)
    <86>   DW_AT_external    : 1
    <86>   DW_AT_name        : (indirect string, offset: 0x0): func
    <8a>   DW_AT_decl_file   : 1
    <8b>   DW_AT_decl_line   : 1
    <8c>   DW_AT_decl_column : 5
    <8d>   DW_AT_prototyped  : 1
    <8d>   DW_AT_type        : <0x6d>
    <91>   DW_AT_low_pc      : 0x1129
    <99>   DW_AT_high_pc     : 0x31
    <a1>   DW_AT_frame_base  : 1 byte block: 9c         (DW_OP_call_frame_cfa)
    <a3>   DW_AT_call_all_calls: 1
```
- Function func() starts at 0x1129, size of 0x31 bytes.

:::::::::::::: {.columns}
::: {.column width=50%}

2. Find cfa offsets within func().
```sh
Contents of the .eh_frame section:

00000070 000000000000001c 00000074 FDE cie=00000000 pc=0000000000001129..000000000000115a
  DW_CFA_advance_loc: 5 to 000000000000112e
  DW_CFA_def_cfa_offset: 16
  DW_CFA_offset: r6 (rbp) at cfa-16

  DW_CFA_advance_loc: 3 to 0000000000001131
  DW_CFA_def_cfa_register: r6 (rbp)

  DW_CFA_advance_loc: 40 to 0000000000001159
  DW_CFA_def_cfa: r7 (rsp) ofs 8
```
- Since 0x112e, cfa offset is 16 (DW_CFA_def_cfa_offset).

:::
::: {.column width=50%}

3. Find cfa offset at the target instruction.
```sh
$ objdump --disassemble main

0000000000001129 <func>:
    1129:       f3 0f 1e fa             endbr64 
    112d:       55                      push   %rbp
    112e:       48 89 e5                mov    %rsp,%rbp
    1131:       89 7d ec                mov    %edi,-0x14(%rbp)
    1134:       89 75 e8                mov    %esi,-0x18(%rbp)
    1137:       c7 45 f0 aa aa aa aa    movl   $0xaaaaaaaa,-0x10(%rbp)
    113e:       c7 45 f4 bb bb bb bb    movl   $0xbbbbbbbb,-0xc(%rbp)
    1145:       c7 45 f8 cc cc cc cc    movl   $0xcccccccc,-0x8(%rbp)
    114c:       c7 45 fc dd dd dd dd    movl   $0xdddddddd,-0x4(%rbp)
    1153:       b8 00 00 00 00          mov    $0x0,%eax
    1158:       5d                      pop    %rbp
    1159:       c3                      ret
```
- The breakpoint will be put at 0x1153. The cfa offset at this instruction is 16.

:::
::::::::::::::


:::::::::::::: {.columns}
::: {.column width=50%}

4. Find DW_AT_location of arguments.
```sh
 <2><a3>: Abbrev Number: 1 (DW_TAG_formal_parameter)
    <a4>   DW_AT_name        : (indirect string, offset: 0x5): argv1
    <a8>   DW_AT_decl_file   : 1
    <a8>   DW_AT_decl_line   : 1
    <a9>   DW_AT_decl_column : 14
    <aa>   DW_AT_type        : <0x6d>
    <ae>   DW_AT_location    : 2 byte block: 91 5c      (DW_OP_fbreg: -36)

 <2><b1>: Abbrev Number: 1 (DW_TAG_formal_parameter)
    <b2>   DW_AT_name        : (indirect string, offset: 0x9e): argv2
    <b6>   DW_AT_decl_file   : 1
    <b6>   DW_AT_decl_line   : 1
    <b7>   DW_AT_decl_column : 25
    <b8>   DW_AT_type        : <0x6d>
    <bc>   DW_AT_location    : 2 byte block: 91 58      (DW_OP_fbreg: -40)
```
- DW_AT_location of argv1 is (DW_OP_fbreg: -36).
- DW_AT_location of argv2 is (DW_OP_fbreg: -40).

:::
::: {.column width=50%}

5. Calculate argument addresses.
```sh

argv1 addr  = DW_AT_location + CFA_Offset
            = (DW_OP_fbreg - 36) + CFA_Offset
            = (rbp - 36) + 16
            = rbp - 20


argv2 addr  = DW_AT_location + CFA_Offset
            = (DW_OP_fbreg - 40) + CFA_Offset
            = (rbp - 40) + 16
            = rbp - 24
  
```

- argv1 will locate at rbp - 20.
- argv2 will locate at rbp - 24.

:::
::::::::::::::

## Print the Argument Values

:::::::::::::: {.columns}
::: {.column width=50%}

1. Set breakpoint.
```sh
(gdb) break *func+42
Breakpoint 1 at 0x1153: file main.c, line 7.

(gdb) run
Breakpoint 1, func (argv1=286331153, argv2=572662306) at main.c:7

(gdb) disassemble
Dump of assembler code for function func:
   0x0000555555555129 <+0>:     endbr64 
   0x000055555555512d <+4>:     push   %rbp
   0x000055555555512e <+5>:     mov    %rsp,%rbp
   0x0000555555555131 <+8>:     mov    %edi,-0x14(%rbp)
   0x0000555555555134 <+11>:    mov    %esi,-0x18(%rbp)
   0x0000555555555137 <+14>:    movl   $0xaaaaaaaa,-0x10(%rbp)
   0x000055555555513e <+21>:    movl   $0xbbbbbbbb,-0xc(%rbp)
   0x0000555555555145 <+28>:    movl   $0xcccccccc,-0x8(%rbp)
   0x000055555555514c <+35>:    movl   $0xdddddddd,-0x4(%rbp)
=> 0x0000555555555153 <+42>:    mov    $0x0,%eax
   0x0000555555555158 <+47>:    pop    %rbp
   0x0000555555555159 <+48>:    ret
```

:::
::: {.column width=50%}

2. Print function arguments.
```sh
# print argv1
(gdb) x/wx $rbp-20
0x7fffffffe5bc: 0x11111111

# print argv2
(gdb) x/wx $rbp-24
0x7fffffffe5b8: 0x22222222
```

:::
::::::::::::::

# GDB Utilities

:::::::::::::: {.columns}
::: {.column width=50%}

Print frame info.
```sh
(gdb) info frame
Stack level 0, frame at 0x7fffffffe5e0:
 rip = 0x555555555153 in func (main.c:7); saved rip = 0x55555555517c
 called by frame at 0x7fffffffe600
 source language c.
 Arglist at 0x7fffffffe5d0, args: argv1=286331153, argv2=572662306
 Locals at 0x7fffffffe5d0, Previous frame's sp is 0x7fffffffe5e0
 Saved registers:
  rbp at 0x7fffffffe5d0, rip at 0x7fffffffe5d8
```

:::
::: {.column width=50%}

Print function arguments and local variables.
```sh
(gdb) info args
argv1 = 286331153
argv2 = 572662306

(gdb) info locals
a = -1431655766
b = -1145324613
c = -858993460
d = -572662307
```

:::
::::::::::::::

