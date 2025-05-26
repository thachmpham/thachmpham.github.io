---
title: "Inside the Call Stack"
subtitle: "**Examining C Function Calls at the x86 Assembly Level**"
---


# 1. Prolog & Epilog
**Prolog** is a the code at the beginning of a function, which prepare the stack and registers for use within the function.  

Below is a typical example of a function prolog.
```nasm
  
push   %rbp        ; Save the old base pointer on the stack
mov    %rsp, %rbp  ; Set the new base pointer to the current stack pointer
  
```
- `rbp`: *Base pointer, the base of the current stack frame. Provides a stable reference point for accessing function parameters and local variables.*

**Epilog** is a the code at the end of the function, undo what the prolog did, which restores the stack and registers to the state they were in before the function was called.

Below is a typical example of a function epilog.
```nasm
  
mov    $0x1, %eax   ; Set return value to 1 (in %eax)
pop    %rbp         ; Restore previous base pointer
ret                 ; Return to caller
  
```

In the **demo** below, we will compile a simple function with gcc and then disassemble it using objdump.
```c
  
// file: function.c

int func_empty()
{
    return 1;
}
  
```

```sh
  
$ gcc -O0 -c function.c

$ objdump --disassemble function.o
  
```

```nasm
  
0000000000000000 <func_empty>:
   0:   f3 0f 1e fa             endbr64
   4:   55                      push   %rbp
   5:   48 89 e5                mov    %rsp,%rbp
   8:   b8 01 00 00 00          mov    $0x1,%eax
   d:   5d                      pop    %rbp
   e:   c3                      ret
  
```


# 2. Arguments & Local Variables
**Local variables** live on the stack for each function call, and their addresses are based on negative offsets from the base pointer `rbp`.  

:::::::::::::: {.columns}
::: {.column}
**C++**
```c
int a, b, c;
a = 1;
b = 2;
c = 3;
```
:::
::: {.column}
**x86-64 Assembly**
```nasm
sub    $0x20,%rsp       ; Reserve 32 bytes on the stack
movl   $0x1,-0xc(%rbp)  ; Variable a located at $rbp-0xc
movl   $0x2,-0x8(%rbp)  ; Variable b located at $rbp-0x8
movl   $0x3,-0x4(%rbp)  ; Variable c located at $rbp-0x4
```
:::
::::::::::::::

**Function arguments** are passed via registers first, specifically `rdi, rsi, rdx, rcx, r8, r9`. If there are more than 6 arguments, the extra ones are passed on the stack. their addresses are based on negative offsets from the base pointer `rbp`.

:::::::::::::: {.columns}
::: {.column}
**C++**
```c

func(a, b);
```
:::
::: {.column}
**x86-64 Assembly**
```nasm
; Save data to registers before call function
  mov    -0x8(%rbp),%esi  ; $esi = b
  mov    -0xc(%rbp),%edi  ; $edi = a
  call   55 <main+0x37>   ; call func()

<func>:
; Get data from register after enter function
  mov    %edi,-0x14(%rbp) ; argv1 = $edi = a
  mov    %esi,-0x18(%rbp) ; argv2 = $esi = b
```
:::
::::::::::::::

In the **demo** below, we will examine a simple program that calls sum function.
```c
  
// file: main.c

int sum(int argv1, int argv2)
{
    int s = argv1 + argv2;
    return s;
}

int main(int argc, char** argv)
{    
    int a = 1;
    int b = 2;
    int c = 3;

    sum(a, b);

    return 0;
}
  
```

```sh
  
$ gcc -O0 -o main main.c

$ objdump --disassemble --no-show-raw-insn main
  
```

```nasm
  
000000000000001e <main>:
  1e:   endbr64
  22:   push   %rbp             
  23:   mov    %rsp,%rbp        
  26:   sub    $0x20,%rsp       ; Revere 32 bytes for local variables
  2a:   mov    %edi,-0x14(%rbp) ; Command-line argument: argc = $edi
  2d:   mov    %rsi,-0x20(%rbp) ; Command-line argument: argv = $rsi
  31:   movl   $0x1,-0xc(%rbp)  ; a = 1
  38:   movl   $0x2,-0x8(%rbp)  ; b = 2
  3f:   movl   $0x3,-0x4(%rbp)  ; c = 3
  46:   mov    -0x8(%rbp),%edx  ; $edx = b = 2
  49:   mov    -0xc(%rbp),%eax  ; $eax = a = 1
  4c:   mov    %edx,%esi        ; $esi = $edx = b = 2
  4e:   mov    %eax,%edi        ; $edi = $eax = a = 1
  50:   call   55 <main+0x37>   ; call sum()
  55:   mov    $0x0,%eax
  5a:   leave
  5b:   ret
  

0000000000000000 <sum>:
   0:   endbr64
   4:   push   %rbp             ; Prolog
   5:   mov    %rsp,%rbp        ; Prolog
   8:   mov    %edi,-0x14(%rbp) ; argv1 = $edi = a = 1
   b:   mov    %esi,-0x18(%rbp) ; argv2 = $esi = b = 2
   e:   mov    -0x14(%rbp),%edx ; $edx = argv1 = 1
  11:   mov    -0x18(%rbp),%eax ; $eax = argv2 = 2
  14:   add    %edx,%eax        ; $eax = $eax + $edx = 2 + 1 = 3
  16:   mov    %eax,-0x4(%rbp)  ; c = $eax = 3
  19:   mov    -0x4(%rbp),%eax  ; Epilog
  1c:   pop    %rbp             ; Epilog
  1d:   ret                     ; Epilog
  
```


# 2. Examine Call Stack
## 2.1. Arguments & Local Variables
Examine the memory locations of function arguments and local variables by using the **rbp** register as a reference point.

```sh
  
$ gdb main

(gdb) break sum
Breakpoint 1 at 0x1131

(gdb) run
Starting program: /root/demo/main 
Breakpoint 1, 0x0000555555555131 in sum ()

(gdb) disassemble
Dump of assembler code for function sum:
   0x0000555555555129 <+0>:     endbr64
   0x000055555555512d <+4>:     push   %rbp
   0x000055555555512e <+5>:     mov    %rsp,%rbp
=> 0x0000555555555131 <+8>:     mov    %edi,-0x14(%rbp) ; argv1
   0x0000555555555134 <+11>:    mov    %esi,-0x18(%rbp) ; argv2
   0x0000555555555137 <+14>:    mov    -0x14(%rbp),%edx
   0x000055555555513a <+17>:    mov    -0x18(%rbp),%eax
   0x000055555555513d <+20>:    add    %edx,%eax
   0x000055555555513f <+22>:    mov    %eax,-0x4(%rbp)  ; variable s
   0x0000555555555142 <+25>:    mov    -0x4(%rbp),%eax
   0x0000555555555145 <+28>:    pop    %rbp
   0x0000555555555146 <+29>:    ret
End of assembler dump.

(gdb) break *(sum+28)
Breakpoint 2 at 0x555555555145

(gdb) continue
Continuing.
Breakpoint 2, 0x0000555555555145 in sum ()

(gdb) x/24bx $rbp-24
0x7fffffffde08: 0x02    0x00    0x00    0x00    0x01    0x00    0x00    0x00
0x7fffffffde10: 0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
0x7fffffffde18: 0x00    0x00    0x00    0x00    0x03    0x00    0x00    0x00
  
```

Due to little-endian order, 4-byte integers are stored in memory with the least significant byte first, which is the reverse of how we usually write numbers.

- Function argument argv2 = 2 is stored as {0x02, 0x00, 0x00, 0x00}.
- Function argument argv1 = 1 as {0x01, 0x00, 0x00, 0x00}
- The local variable s = 3 as {0x03, 0x00, 0x00, 0x00}.

<pre class="mermaid">
block-beta  
    columns 8
    argv2["argv2"]:4 argv1["argv1"]:4
    pad1["padding"]:8
    pad2["padding"]:4 s["s"]:4
</pre>


## 2.1. Call chain
Examine the call chain with by using the **rip** and **rsp** registers.
```sh
  
// file: main.c

void function_0();
void function_1();
void function_2();

int main(int argc, char** argv)
{    
    function_0();
    return 0;
}

void function_0()
{
    function_1();
}

void function_1()
{
    function_2();
}

void function_2()
{
    
}
  
```

```sh
  
$ gcc -O0 -o main main.c
  
```

```sh
  
$ gdb main

(gdb) break function_2
Breakpoint 1 at 0x117f

(gdb) run
Starting program: /root/demo/main 
Breakpoint 1, 0x000055555555517f in function_2 ()

(gdb) bt
#0  0x000055555555517f in function_2 ()
#1  0x0000555555555174 in function_1 ()
#2  0x000055555555515f in function_0 ()
#3  0x0000555555555146 in main ()

(gdb) x/a $rip
0x55555555517f <function_2+8>:  0x1e0ff30000c35d90

(gdb) x/10a $rsp
0x7fffffffde10: 0x7fffffffde20  0x555555555174 <function_1+18>
0x7fffffffde20: 0x7fffffffde30  0x55555555515f <function_0+18>
0x7fffffffde30: 0x7fffffffde50  0x555555555146 <main+29>
0x7fffffffde40: 0x7fffffffdf78  0x1ffffdf78
0x7fffffffde50: 0x7fffffffdef0  0x7ffff7c2a1ca <__libc_start_call_main+122>
  
```

<pre class="mermaid">
flowchart RL
    subgraph rip
        function_2+8
    end   

   subgraph rsp
        function_1+18
        function_0+18              
        main+29
        __libc_start_call_main+122
    end 

    __libc_start_call_main+122 --> main+29 --> function_0+18 --> function_1+18 --> function_2+8
</pre>