---
title: "GDB: DebugInfo"
---


## Binary with DebugInfo

:::::::::::::: {.columns}
::: {.column width=70%}

Build binary with debug info.
```sh
  gcc -g ...
  g++ -g ...
```

List sections using objdump.
```sh
  objdump --section-headers binary
```

List sections using gdb.
```sh
  maint info [-all-objects] sections [filters]
```

- *-all-objects*: info of all object files.
- *filters*: section name, flag.

<br>

Sample: Build binary with debuginfo.

```c 
int sum(int a, int b)
{
    return a + b;
}

int main()
{
    sum(1, 2);
}
  
```

```sh
#-------------------------------
$ gcc -g demo.c -o demo

$ objdump --section-headers demo
#-------------------------------
  
```

:::
::: {.column width=30%}

Binary with debuginfo.
```yml
0.  note.gnu.build-id
1.  interp
2.  gnu.hash
3.  dynsym
4.  dynstr
5.  gnu.version
6.  gnu.version_r
7.  rela.dyn
8.  rela.plt
9.  init
10. plt
11. text
12. fini
13. rodata
14. eh_frame_hdr
15. eh_frame
16. note.ABI-tag
17. init_array
18. fini_array
19. dynamic
20. got
21. got.plt
22. data
23. bss
24. comment
25. annobin.notes
26. gnu.build.attributes
#-----------------------
27. debug_aranges
28. debug_info
29. debug_abbrev
30. debug_line
31. debug_str
32. debug_line_str
  
```

:::
::::::::::::::


## Split DebugInfo

:::::::::::::: {.columns}
::: {.column width=70%}

GDB allows to split the debuginfo of a binary into a separate file. The purpose is to optimize binary size. When debugging is needed later, the debug information can be reattached.

Extract debuginfo into a file.
```sh
  objcopy --only-keep-debug binary debuginfo
```

Remove debuginfo from binary.
```sh
  strip --strip-debug binary
```

<br>

Sample: Strip a binary.
```sh
#-------------------------------
$ gcc -g demo.c -o demo

$ objcopy --only-keep-debug demo demo.debuginfo
$ strip --strip-debug demo

$ objdump --section-headers demo
#-------------------------------
  
```

:::
::: {.column width=30%}

Without debuginfo.
```yml
0.  note.gnu.build-id
1.  interp
2.  gnu.hash
3.  dynsym
4.  dynstr
5.  gnu.version
6.  gnu.version_r
7.  rela.dyn
8.  rela.plt
9.  init
10. plt
11. text
12. fini
13. rodata
14. eh_frame_hdr
15. eh_frame
16. note.ABI-tag
17. init_array
18. fini_array
19. dynamic
20. got
21. got.plt
22. data
23. bss
24. comment
25. annobin.notes
26. gnu.build.attributes
  
```

:::
::::::::::::::


## Attach DebugInfo

:::::::::::::: {.columns}
::: {.column width=70%}

Creates a .gnu_debuglink section which contains a reference to debuginfo file and adds it to the binary file.
```sh
  objcopy --add-gnu-debuglink=debuginfo binary
```

<br>

Sample: Attach debuginfo to binary
```sh
objcopy --add-gnu-debuglink=demo.debuginfo demo
```

:::
::: {.column width=30%}

Without debuginfo.
```yml
0.  note.gnu.build-id
1.  interp
2.  gnu.hash
..  .....
24. comment
25. annobin.notes
26. gnu.build.attributes
#-----------------------
27. gnu_debuglink
  
```

:::
::::::::::::::


## Debuginfo Search Directory

:::::::::::::: {.columns}
::: {.column width=50%}

Default, GDB searches for debuginfo files in the below directories.

- Absolute paths.
- Same directory as binary.
- In .debug subdirectory.
- /usr/lib/debug

:::
::: {.column width=50%}

However, we can change the search directories.
```sh
  set debug-file-directory dirs
  show debug-file-directory
```

- *dirs*: multiple paths separated by ':'.

:::
::::::::::::::