---
title: "Inside the Call Stack"
subtitle: "Exploring C Function Calls via x86 Assembly"
---


# 1. Empty Function
```c
  
void func_empty()
{

}

int main(int argc, char** argv)
{
    func_empty();
    return 0;
}
  
```

```sh
  
$ gcc -O0 -fno-branch-protection -o main main.c

$ objdump --disassemble --no-show-raw-insn main
  
```

```nasm
  
0000000000001129 <func_empty>:
    1129:       push   %rbp             # save old base pointer
    112a:       mov    %rsp,%rbp        # set new base pointer
    112d:       nop                     # no operation
    112e:       pop    %rbp             # restore old base pointer
    112f:       ret                     # return from function

0000000000001130 <main>:
    1130:       push   %rbp             # save old base pointer
    1131:       mov    %rsp,%rbp        # set new base pointer
    1134:       sub    $0x10,%rsp       # allocate 16 bytes on stack
    1138:       mov    %edi,-0x4(%rbp)  # store argc (int) at rbp-4
    113b:       mov    %rsi,-0x10(%rbp) # store argv (char**) at rbp-16
    113f:       mov    $0x0,%eax        # set return value 0
    1144:       call   1129 <func_empty>    # call func_empty
    1149:       mov    $0x0,%eax        # set return value 0
    114e:       leave                   # restore stack frame
    114f:       ret                     # return from main
  
```