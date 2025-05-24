---
title: "Inside the Call Stack"
subtitle: "Exploring C Function Calls via x86 Assembly"
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

