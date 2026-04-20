---
title: "GDB Catchpoint"
subtitle: "Stop on a program event"
---


:::::::::::::: {.columns}
::: {.column}

| Fork |  |
|:-------------|:-------------|
| `catch fork`                | Stop execution on fork |
| `set detach-on-fork on`     | Detach child from gdb (default). |
| `set detach-on-fork off`    | Keep both parent, child under gdb. |
| `show detach-on-fork`       |           |
| `set follow-fork-mode parent` | Switch gdb to parent (default) |
| `set follow-fork-mode child`  | Switch gdb to child |
| `show follow-fork-mode`       |         |

| Exec |  |
|:-------------|:-------------|
| `catch exec`                | Stop execution on exec |
| `set follow-exec-mode same` | Keep gdb on the current process |
| `set follow-exec-mode new`  | Switch gdb to the new process |
| `show follow-exec-mode`     |   |

:::
::: {.column}

| Signal |  |
|:-------------|:-------------|
| `catch signal SIGSEGV`      | Stop execution on SIGSEGV |
| `catch signal all`          | Stop execution on all signals |

| Tips & Tricks |  |
|:-------------|:-------------|
| Catch before process exits  | `catch syscall exit` |
|                             | `catch syscall exit_group` |

:::
::::::::::::::
