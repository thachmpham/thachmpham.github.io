---
title: "Memory"
---

# Stack

:::::::::::::: {.columns}
::: {.column width=33%}

**Definition**  
The stack is a per-thread memory region, meaning each thread has its own stack that is not shared with others.

:::
::: {.column width=33%}

**Movement**  
On each function call, the stack grows downward from high address to low address.
It shrinks when function returns

:::
::: {.column width=33%}

**Contents**  
Stores local variables and saved registers, such as the previous %rbp, and the return address (or the next instruction).

:::
::::::::::::::

## Stack Pointer
The stack pointer (rsp) always points to the current top of the stack. The push/pop, call/ret instructions will adjust %rsp.

### Push & Pop
The push instruction put a value onto the stack. The pop instruction remove a value from the stack. Register rsp automatically moves to the top of the stack.

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

Monitor the stack through instructions.
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

Changes of the stack.
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

### Call & Ret
The call instruction pushes the address of the next instruction (rip) onto the stack then jump to the target function. This address is also called the return address.  
The ret instruction pops the return address from the stack and jump to this address.

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

Monitor the stack through instructions.
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

Changes of the stack.
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

- The call instruction pushes $rip 0x0000000000401013 to the stack, and jump to the sum function.
- The ret instruction pops 0x0000000000401013 from the stack, and then jump to this address.

## Base Pointer
The base pointer (rbp) stores the start address of the function stack frame. It remains fixed throughout the function, allowing compiler to use its stable position to generate offsets for local variables and parameter.

### Prolog & Epilog

:::::::::::::: {.columns}
::: {.column width=40%}

When calling a new C/C++ function, prolog is the compiler-generated code that sets up the stack frame for the new function.

When leaving a C/C++ function, epilog is the compiler-generated code that restores the parent's stack frame and jump back to parent function.

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

Changes of the stack on prolog and epilog.
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

### Arguments & Local Variables

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

## Examine Stack Frame

:::::::::::::: {.columns}
::: {.column width=50%}

The call instruction pushes the parent $rip onto the stack. Prolog pushes the parent $rip onto the stack.
Function arguments and local variables are stored in the stack, and their addresses are based on the current $rbp.

As you can see in the figure:

- From current $rbp, go backward to lower addresses, we reach the local variables and arguments.
- From current $rbp, go forward to higher addresses, we reach the parent $rbp and $rip.

So, in GDB:

- To examine arguments and local variables, examine backward from RBP, eg: `x/-6wx $rbp`.
- To examine the parent’s RBP and RIP, examine forward from RBP: `x/2a $rbp`.

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

File main.c
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
$ gcc main.c -o main
```

Run under GDB.

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

Set breakpoint at the end of func.

```sh
(gdb) break *func+42

(gdb) run
Breakpoint 1, 0x0000555555555153 in func ()
```

Examine arguments and local variables.
```sh
(gdb) x/-6wx $rbp
0x7fffffffe5b8: 0x22222222      0x11111111      0xaaaaaaaa      0xbbbbbbbb
0x7fffffffe5c8: 0xcccccccc      0xdddddddd
```

Examine parent's $rip and $rbp.
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

# GDB
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

Sample: Add a memory region by gdb.

:::::::::::::: {.columns}
::: {.column}

- main.c
```c
int main()
{
  return 0
}
```

:::
::: {.column}

- hello.txt
```sh
hello world
```

:::
::::::::::::::

- GDB session.

```sh
(gdb) set $fd = (int) open("hello.txt", 0)

(gdb) set $region_addr = (void*) mmap(0, 4096, 1, 1, $fd, 0)

(gdb) info proc mappings
Start Addr         End Addr           Size    Offset  Perms File 
0x0000fffff7ff5000 0x0000fffff7ff6000 0x1000  0x0     r--s  /root/demo/hello.txt 
...

(gdb) x/s $region_addr
0xfffff7ff5000: "hello world\n"
    
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