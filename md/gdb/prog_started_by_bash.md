---
title: 'Debug Programs Started By Bash Script'
---

:::::::::::::: {.columns}
::: {.column width=50%}

1. Script run.sh starts ls program.
```sh
ls /home
```

2. Start script under gdb.
```sh
$ gdb --args bash run.sh
```

:::
::: {.column width=50%}

3. Configure gdb.
```sh
(gdb) set detach-on-fork off
(gdb) set follow-fork-mode child
(gdb) catch exec
```

4. Run debug.
```sh
(gdb) run
[New inferior 2 (process 5911)]
process 5911 is executing new program: /usr/bin/ls
[Switching to process 5911]
Thread 2.1 "ls" hit Catchpoint 1 (exec'd /usr/bin/ls')

(gdb) info inferiors 
  Num  Description       Connection           Executable        
  1    process 5909      1 (native)           /usr/bin/bash     
* 2    process 5911      1 (native)           /usr/bin/ls
```

:::
::::::::::::::