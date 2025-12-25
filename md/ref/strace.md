---
title: "strace"
---

## Filter

:::::::::::::: {.columns}
::: {.column width=40%}

| Syscall | Example |
|:--------|:--------|
| `-e trace=syscall`        | `-e trace=read`                   |
|                           | `-e trace='!read'`                |
| `-e trace=syscalls`       | `-e trace=read,write`             |
| `-e trace=syscall_set`    | `-e trace=file`                   |
|                           | `-e trace=file -e trace='!write'` |
| `-e trace=/regex`         | `-e trace=/open.*`                |

:::
::: {.column width=30%}

| File Descriptor | Example      |
|:----------------|:-------------|
| `-e fds=set`    | `-e fds=1`   |
|                 | `-e fds=!1`  |
|                 | `-e fds=1,2` |

| Trace path      | Example      |
|:----------------|:-------------|
| `--trace-path=path` | `-trace-path=hello` |

:::
::: {.column width=30%}

| Return Status   | Value      |
|:----------------|:-------------|
| `-e status=set` | `successful, failed, unfinished` |
|                 | `detached, unavailable` |

| Signal          | Example      |
|:----------------|:-------------|
| `-e signals=set` | `-e signals=SIGIO` |

:::
::::::::::::::

## Display

:::::::::::::: {.columns}
::: {.column width=60%}

| Decode Argument       | Example       | Explain |
|:----------------------|:------------- |:---------------------------------------|
| `--write=fd_set`      | `--write=1`   | Print arguments of write on fd 1 in hex + ascii |
| `--read=fd_set`       | `--read=0`    | Print arguments of read on fd 0 in hex + ascii |
| `xx -s N`             | `-xx -s 32`   | Print arguments in hex, max length is 32 |

| Decode Descriptor     | Example               | Explain |
|:----------------------|:----------------------|:------------------------------------|
| `--decode-fds`        |                       |                                     |
| `--decode-fds=set`    | `--decode-fds=path`   | Print file path associated with fds |
|                       | `--decode-fds=socket` | Print protocol associated with fds  |
| `--decode-pids=set`   | `--decode-pids=comm`  | Print command names for pids        |

:::
::: {.column width=40%}

| Fork & Daemon                     | Explain                            |
|:----------------------------------|:-----------------------------------|
| `-o myfile`                       | Output to myfile                   |
| `-o myfile --output-separately`   | Output to file myfile.pid          |
| `-o myfile -f`                    | Follow child, output to myfile     |
| `-o myfile -ff`                   | Follow child, output to myfile.pid |
| `-o myfile -D`                    | strace and target process detached |

| Backtrace              |
|:-----------------------|
| `--stack-trace=symbol` |
| `--stack-trace=source` |

:::
::::::::::::::
