---
title: 'Reverse the Stack Frame'
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

# Raw Memory of Stack Frame

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


# DWARF
## Canonical Frame Address (CFA)
A Canonical Frame Address (CFA) is a fixed address in a stack frame. CFA is used as a pivot to locate return addresses, function arguments and local variables.

### CFA Rules in ELF
CFA rules define how to compute the Canonical Frame Address at each instruction in a function.

:::::::::::::: {.columns}
::: {.column width=50%}

1. Print CFA rules.
```sh
$ readelf --debug-dump=frames-interp main
Contents of the .eh_frame section:
00000070 000000000000001c 00000074 FDE cie=00000000 pc=0000000000001129..000000000000115a
   LOC           CFA      rbp   ra
0000000000001129 rsp+8    u     c-8
000000000000112e rsp+16   c-16  c-8
0000000000001131 rbp+16   c-16  c-8
0000000000001159 rsp+8    c-16  c-8
```

:::
::: {.column width=50%}

In the below objdump output, range of func() is from `0x1129` to `0x1159`. Use this range to find the corresponding block in readelf output, we have `pc=0000000000001129..000000000000115a`.

So, CFA rules in func():

- `1129: cfa = rsp + 8`
- `112e: cfa = rsp + 16`
- `1131: cfa = rbp + 16`
- `1159: cfa = rsp + 8`

:::
::::::::::::::

:::::::::::::: {.columns}
::: {.column width=50%}

2. Disassemble function func().
```sh
$ objdump --disassemble --no-show-raw-insn main
0000000000001129 <func>:
    1129:       endbr64
    112d:       push   %rbp
    112e:       mov    %rsp,%rbp
    1131:       mov    %edi,-0x14(%rbp)
    1134:       mov    %esi,-0x18(%rbp)
    1137:       movl   $0xaaaaaaaa,-0x10(%rbp)
    113e:       movl   $0xbbbbbbbb,-0xc(%rbp)
    1145:       movl   $0xcccccccc,-0x8(%rbp)
    114c:       movl   $0xdddddddd,-0x4(%rbp)
    1153:       mov    $0x0,%eax
    1158:       pop    %rbp
    1159:       ret
```

:::
::: {.column width=50%}

Line-by-line explanation.
```sh

0000000000001129 <func>:
    1129:   initially, assign cfa:                          cfa = rsp + 8
    112d:   push instruction moves stack pointer down:      rsp = rsp - 8
    112e:   to keep cfa point to the assigned address:      cfa = rsp + 16
    1131:   switch cfa rule to base pointer:                cfa = rbp + 16
    1134:
    1137:
    113e:
    1145:
    114c:
    1153:
    1158:   pop instruction moves stack pointer up:         rsp = rsp + 8
    1159:   to keep cfa point to the assigned address:      cfa = rsp + 8
```

:::
::::::::::::::



### Absolute CFA at Runtime

:::::::::::::: {.columns}
::: {.column width=50%}

1. Set a breakpoint at the begin of func().
```sh
(gdb) set print frame-info location-and-address

(gdb) break *func+0
Breakpoint 1 at 0x1129: file main.c, line 2.

(gdb) run
Starting program: /root/demo/main
Breakpoint 1, 0x0000555555555129 in func (argv1=0, argv2=0) at main.c:2
```

2. Calculate assigned address of CFA.
```sh
(gdb) disassemble
Dump of assembler code for function func:
=> 0x0000555555555129 <+0>:     endbr64
   0x000055555555512d <+4>:     push   %rbp
   0x000055555555512e <+5>:     mov    %rsp,%rbp
   0x0000555555555131 <+8>:     mov    %edi,-0x14(%rbp)
   0x0000555555555134 <+11>:    mov    %esi,-0x18(%rbp)
   0x0000555555555137 <+14>:    movl   $0xaaaaaaaa,-0x10(%rbp)
   0x000055555555513e <+21>:    movl   $0xbbbbbbbb,-0xc(%rbp)
   0x0000555555555145 <+28>:    movl   $0xcccccccc,-0x8(%rbp)
   0x000055555555514c <+35>:    movl   $0xdddddddd,-0x4(%rbp)
   0x0000555555555153 <+42>:    mov    $0x0,%eax
   0x0000555555555158 <+47>:    pop    %rbp
   0x0000555555555159 <+48>:    ret
End of assembler dump.

(gdb) print $rsp
$1 = (void *) 0x7fffffffe5d8

(gdb) print $rsp + 8
$2 = (void *) 0x7fffffffe5e0
```
- According to CFA rules in readelf output, at func+0,
    - `cfa = rsp + 8 = 0x7fffffffe5e0`


3. Watch rsp, rbp.
```sh
(gdb) watch $rsp
Watchpoint 2: $rsp

(gdb) watch $rbp
Watchpoint 3: $rbp
```


4. Check CFA after rsp changes.
```sh
(gdb) continue
Continuing.

Watchpoint 2: $rsp

Old value = (void *) 0x7fffffffe5d8
New value = (void *) 0x7fffffffe5d0
0x000055555555512e in func (argv1=0, argv2=0) at main.c:2
```
After push instruction, rsp moves down from `0x7fffffffe5d8` to `0x7fffffffe5d0`.


```sh
(gdb) disassemble
Dump of assembler code for function func:
   0x0000555555555129 <+0>:     endbr64
   0x000055555555512d <+4>:     push   %rbp
=> 0x000055555555512e <+5>:     mov    %rsp,%rbp
   0x0000555555555131 <+8>:     mov    %edi,-0x14(%rbp)
   0x0000555555555134 <+11>:    mov    %esi,-0x18(%rbp)
   0x0000555555555137 <+14>:    movl   $0xaaaaaaaa,-0x10(%rbp)
   0x000055555555513e <+21>:    movl   $0xbbbbbbbb,-0xc(%rbp)
   0x0000555555555145 <+28>:    movl   $0xcccccccc,-0x8(%rbp)
   0x000055555555514c <+35>:    movl   $0xdddddddd,-0x4(%rbp)
   0x0000555555555153 <+42>:    mov    $0x0,%eax
   0x0000555555555158 <+47>:    pop    %rbp
   0x0000555555555159 <+48>:    ret
End of assembler dump.

(gdb) print $rsp + 16
$3 = (void *) 0x7fffffffe5e0
```
- According to CFA rules, at func+5,
    - `cfa = rsp + 16 = 0x7fffffffe5e0`

:::
::: {.column width=50%}

5. Check CFA after rbp changes.
```sh
(gdb) continue
Continuing.

Watchpoint 3: $rbp

Old value = (void *) 0x7fffffffe5f0
New value = (void *) 0x7fffffffe5d0
0x0000555555555131 in func (argv1=0, argv2=0) at main.c:2
```
The rbp changes from `0x7fffffffe5f0` to `0x7fffffffe5d0`.


```sh
(gdb) disassemble
Dump of assembler code for function func:
   0x0000555555555129 <+0>:     endbr64
   0x000055555555512d <+4>:     push   %rbp
   0x000055555555512e <+5>:     mov    %rsp,%rbp
=> 0x0000555555555131 <+8>:     mov    %edi,-0x14(%rbp)
   0x0000555555555134 <+11>:    mov    %esi,-0x18(%rbp)
   0x0000555555555137 <+14>:    movl   $0xaaaaaaaa,-0x10(%rbp)
   0x000055555555513e <+21>:    movl   $0xbbbbbbbb,-0xc(%rbp)
   0x0000555555555145 <+28>:    movl   $0xcccccccc,-0x8(%rbp)
   0x000055555555514c <+35>:    movl   $0xdddddddd,-0x4(%rbp)
   0x0000555555555153 <+42>:    mov    $0x0,%eax
   0x0000555555555158 <+47>:    pop    %rbp
   0x0000555555555159 <+48>:    ret
End of assembler dump.

(gdb) print $rbp + 16
$4 = (void *) 0x7fffffffe5e0
```
- According to CFA rules, at func+8,
    - `cfa = rbp + 16 = 0x7fffffffe5e0`


6. Check CFA after rsp changes.
```sh
(gdb) continue
Continuing.

Watchpoint 2: $rsp

Old value = (void *) 0x7fffffffe5d0
New value = (void *) 0x7fffffffe5d8

Watchpoint 3: $rbp

Old value = (void *) 0x7fffffffe5d0
New value = (void *) 0x7fffffffe5f0
0x0000555555555159 in func (argv1=286331153, argv2=572662306) at main.c:8
```
After pop instruction, rsp changes from `0x7fffffffe5d0` to `0x7fffffffe5d8`.


```sh
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
   0x0000555555555153 <+42>:    mov    $0x0,%eax
   0x0000555555555158 <+47>:    pop    %rbp
=> 0x0000555555555159 <+48>:    ret
End of assembler dump.

(gdb) print $rsp + 8
$5 = (void *) 0x7fffffffe5e0
```
- According to CFA rules, at func+48,
    - `cfa = rsp + 8 = 0x7fffffffe5e0`

<br>

So, within func(), when rsp or rbp changes, the CFA rule is adjusted to keep CFA point to the fixed assigned address `0x7fffffffe5e0`.

:::
::::::::::::::


## Debugging Information Entry (DIE)
The Debugging Information Entry (DIE) is used to translate assembly code back to C/C++ code.

Each DIE entity consists of:

- Tag (What entity describe)
    - DW_TAG_subprogram: Function.
    - DW_TAG_base_type: Data type.
    - DW_TAG_formal_parameter: Function argument.
    - DW_TAG_variable: Global or local variable.
- Attributes (Key-Value pairs):
    - DW_AT_name: Attribute name.
    - DW_AT_type: Attribute type.
    - DW_AT_location: Attribute location.
    - DW_AT_low_pc: Low address (program counter) of a function.

```sh
$ readelf --debug-dump=info main
 <1><6d>: Abbrev Number: 6 (DW_TAG_base_type)                           # Offset 6d, entry describes a data type.
    <6e>   DW_AT_byte_size   : 4                                        # Size: 4
    <70>   DW_AT_name        : int                                      # Data type name is int

 <1><85>: Abbrev Number: 8 (DW_TAG_subprogram)                          # Level 1, entry describes a function.
    <86>   DW_AT_name        : (indirect string, offset: 0x0): func     # Name: func().
    <91>   DW_AT_low_pc      : 0x1129                                   # Low address: 0x1129.
    <99>   DW_AT_high_pc     : 0x31                                     # Length: 0x31.

 <2><a3>: Abbrev Number: 1 (DW_TAG_formal_parameter)                    # Level 2, entry describes a function argument of func().
    <a4>   DW_AT_name        : (indirect string, offset: 0x5): argv1    # Name: argv1
    <aa>   DW_AT_type        : <0x6d>                                   # Type is defined at offset 0x6d, which is int.
    <ae>   DW_AT_location    : 2 byte block: 91 5c (DW_OP_fbreg: -36)   # Frame-based location, location = CFA - 36.

 <2><bf>: Abbrev Number: 2 (DW_TAG_variable)                            # Level 2, entry describes a local variable within func().
    <c0>   DW_AT_name        : a                                        # Name: a
    <c3>   DW_AT_type        : <0x6d>                                   # Type is defined at offset 0x6d, which is int.
    <c7>   DW_AT_location    : 2 byte block: 91 60 (DW_OP_fbreg: -32)   # Frame-based location, location = CBA - 32
```

# Reverse Function Arguments

To reverse a function argument or a local variable at an instruction:

- Find the CFA at the target instruction.
- Find DW_AT_location of the argument/variable.
- Calculate argument/variable address based on CFA and DW_AT_location.
- Dump memory at the address.

<br>

Apply the procedure to reverse the argv1 and argv2 arguments of func().

:::::::::::::: {.columns}
::: {.column width=50%}

1. Find the CFA at the target instruction.

- The breakpoint will be put at 0x1153.
```nasm
$ objdump --disassemble --no-show-raw-insn main
0000000000001129 <func>:
    1129:       endbr64
    112d:       push   %rbp
    112e:       mov    %rsp,%rbp
    1131:       mov    %edi,-0x14(%rbp)
    1134:       mov    %esi,-0x18(%rbp)
    1137:       movl   $0xaaaaaaaa,-0x10(%rbp)
    113e:       movl   $0xbbbbbbbb,-0xc(%rbp)
    1145:       movl   $0xcccccccc,-0x8(%rbp)
    114c:       movl   $0xdddddddd,-0x4(%rbp)
==> 1153:       mov    $0x0,%eax
    1158:       pop    %rbp
    1159:       ret
```

- 0x1153 within range [0x1131, 0x1159). The CFA in this range: cfa = rbp + 16.
```sh
$ readelf --debug-dump=frames-interp main
Contents of the .eh_frame section:
00000070 000000000000001c 00000074 FDE cie=00000000 pc=0000000000001129..000000000000115a
   LOC           CFA      rbp   ra
0000000000001129 rsp+8    u     c-8
000000000000112e rsp+16   c-16  c-8
0000000000001131 rbp+16   c-16  c-8    <==
0000000000001159 rsp+8    c-16  c-8
```
:::
::: {.column width=50%}

2. Find DW_AT_location of the argv1 and argv2 arguments.
```sh
$ readelf --debug-dump=info main
Contents of the .debug_info section:
 <1><85>: Abbrev Number: 8 (DW_TAG_subprogram)
    <86>   DW_AT_name        : (indirect string, offset: 0x0): func
    <91>   DW_AT_low_pc      : 0x1129
 <2><a3>: Abbrev Number: 1 (DW_TAG_formal_parameter)
    <a4>   DW_AT_name        : (indirect string, offset: 0x5): argv1
    <ae>   DW_AT_location    : 2 byte block: 91 5c 	(DW_OP_fbreg: -36)
 <2><b1>: Abbrev Number: 1 (DW_TAG_formal_parameter)
    <b2>   DW_AT_name        : (indirect string, offset: 0x9e): argv2
    <bc>   DW_AT_location    : 2 byte block: 91 58 	(DW_OP_fbreg: -40)
```

3. Calculate argument addresses based on CFA and DW_AT_location.
```python
Location of argv1:
   DW_AT_location = DW_OP_fbreg - 36
                  = cfa - 36
                  = (rbp + 16) - 36
                  = rbp - 20
Location of argv2:
   DW_AT_location = DW_OP_fbreg - 40
                  = cfa - 40
                  = (rbp + 16) - 40
                  = rbp - 24
So,
   argv1 locates at (rbp - 20)
   argv2 locates at (rbp - 24)
```

:::
::::::::::::::

4. Dump memory at the calculated addresses.

:::::::::::::: {.columns}
::: {.column width=50%}

- Run to the target instruction.
```sh
$ gdb main

(gdb) disassemble func
Dump of assembler code for function func:
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

(gdb) break *func+42

(gdb) run
Breakpoint 1, 0x0000555555555153 in func (argv1=286331153, argv2=572662306) at main.c:7
```

:::
::: {.column width=50%}

- Print argv1 and argv2 based on the calculated addresses.
```sh
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

# print argv1
(gdb) x/wx $rbp-20
0x7fffffffe5bc: 0x11111111

# print argv2
(gdb) x/wx $rbp-24
0x7fffffffe5b8: 0x22222222
```

:::
::::::::::::::


# Reverse Local Variables
To reverse local variables, we use the same steps as reverse function arguments.

Apply the procedure to reverse the a,b,c,d variables within func().

:::::::::::::: {.columns}
::: {.column width=50%}

1. Find the CFA at the target instruction.

- The breakpoint will be put at 0x1153.
```nasm
$ objdump --disassemble --no-show-raw-insn main
0000000000001129 <func>:
    1129:       endbr64
    112d:       push   %rbp
    112e:       mov    %rsp,%rbp
    1131:       mov    %edi,-0x14(%rbp)
    1134:       mov    %esi,-0x18(%rbp)
    1137:       movl   $0xaaaaaaaa,-0x10(%rbp)
    113e:       movl   $0xbbbbbbbb,-0xc(%rbp)
    1145:       movl   $0xcccccccc,-0x8(%rbp)
    114c:       movl   $0xdddddddd,-0x4(%rbp)
==> 1153:       mov    $0x0,%eax
    1158:       pop    %rbp
    1159:       ret
```

- 0x1153 within range [0x1131, 0x1159). The CFA in this range: cfa = rbp + 16.
```sh
$ readelf --debug-dump=frames-interp main
Contents of the .eh_frame section:
00000070 000000000000001c 00000074 FDE cie=00000000 pc=0000000000001129..000000000000115a
   LOC           CFA      rbp   ra
0000000000001129 rsp+8    u     c-8
000000000000112e rsp+16   c-16  c-8
0000000000001131 rbp+16   c-16  c-8    <==
0000000000001159 rsp+8    c-16  c-8
```


2. Find DW_AT_location of the a, b, c, d variables.
```sh
$ readelf --debug-dump=info main
Contents of the .debug_info section:

 <1><85>: Abbrev Number: 8 (DW_TAG_subprogram)
    <86>   DW_AT_name        : (indirect string, offset: 0x0): func
    <91>   DW_AT_low_pc      : 0x1129

<2><bf>: Abbrev Number: 2 (DW_TAG_variable)
    <c0>   DW_AT_name        : a
    <c7>   DW_AT_location    : 2 byte block: 91 60 	(DW_OP_fbreg: -32)

 <2><ca>: Abbrev Number: 2 (DW_TAG_variable)
    <cb>   DW_AT_name        : b    
    <d2>   DW_AT_location    : 2 byte block: 91 64 	(DW_OP_fbreg: -28)

 <2><d5>: Abbrev Number: 2 (DW_TAG_variable)
    <d6>   DW_AT_name        : c    
    <dd>   DW_AT_location    : 2 byte block: 91 68 	(DW_OP_fbreg: -24)
    
 <2><e0>: Abbrev Number: 2 (DW_TAG_variable)
    <e1>   DW_AT_name        : d    
    <e8>   DW_AT_location    : 2 byte block: 91 6c 	(DW_OP_fbreg: -20)
```

:::
::: {.column width=50%}

3. Calculate argument addresses based on CFA and DW_AT_location.
```python
Location of a:
   DW_AT_location = DW_OP_fbreg - 32
                  = cfa - 32
                  = (rbp + 16) - 32
                  = rbp - 16
Location of b:
   DW_AT_location = DW_OP_fbreg - 28
                  = cfa - 28
                  = (rbp + 16) - 28
                  = rbp - 12
Location of c:
   DW_AT_location = DW_OP_fbreg - 24
                  = cfa - 24
                  = (rbp + 16) - 24
                  = rbp - 8
Location of d:
   DW_AT_location = DW_OP_fbreg - 20
                  = cfa - 20
                  = (rbp + 16) - 20
                  = rbp - 4
So,
   a locates at (rbp - 16)
   b locates at (rbp - 12)
   c locates at (rbp - 8)
   d locates at (rbp - 4)
```

4. Dump memory at the calculated addresses.

```sh
(gdb) break *func+42

(gdb) run
Breakpoint 1, 0x0000555555555153 in func (argv1=286331153, argv2=572662306) at main.c:7

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

# print variable a
(gdb) x/wx $rbp-16
0x7fffffffe5c0: 0xaaaaaaaa

# print variable b
(gdb) x/wx $rbp-12
0x7fffffffe5c4: 0xbbbbbbbb

# print variable c
(gdb) x/wx $rbp-8
0x7fffffffe5c8: 0xcccccccc

# print variable d
(gdb) x/wx $rbp-4
0x7fffffffe5cc: 0xdddddddd
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

