---
title: "strace"
---

### Filter

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

| Path            | Example      |
|:----------------|:-------------|
| `--trace-path=path` | `--trace-path=hello` |
| `-P`                | `-P hello`           |

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

### Display

:::::::::::::: {.columns}
::: {.column width=60%}

| Decode Argument       | Example       | Explain |
|:----------------------|:------------- |:---------------------------------------|
| `--write=fd_set`      | `--write=1`   | Print data of write on fd 1 in hex + ascii    |
|                       | `--write=all` | Print data of all writes in hex + ascii       |
| `--read=fd_set`       | `--read=0`    | Print data of read on fd 0 in hex + ascii     |
|                       | `--read=all`  | Print data of all reads in hex + ascii        |
| `xx -s N`             | `-xx -s 32`   | Print data in hex, max length is 32           |

| Decode Descriptor     | Abbrev | Explain |
|:----------------------|:-------|:------------------------------------|
| `--decode-fds`        |        |                                     |
| `--decode-fds=set`    |        |                                     |
| `--decode-fds=path`   | `-y`   | Decode file path associated with fds |
| `--decode-fds=all`    | `-yy`  | Decode all info associated with fds  |
|                       |        |                                     |
| `--decode-pids=set`   |        |                                     |
| `--decode-pids=comm`  | `-Y`   | Decode command names for pids        |

:::
::: {.column width=40%}

| Fork & Daemon                     | Explain                            |
|:----------------------------------|:-----------------------------------|
| `-o myfile`                       | Output to myfile                   |
| `-o myfile --output-separately`   | Output to file myfile.pid          |
| `-o myfile -f`                    | Follow child, output to myfile     |
| `-o myfile -ff`                   | Follow child, output to myfile.pid |
| `-o myfile -D`                    | strace and target process detached |

| Backtrace              | Abbrev  |
|:-----------------------|:--------|
| `--stack-trace=symbol` | `-k`    |
| `--stack-trace=source` | `-kk`   |

:::
::::::::::::::


### Real Cases

| Service      | Patch                                                                        |
|:-------------|:-----------------------------------------------------------------------------|
| rsyslog | `ExecStart=strace -yy -o /tmp/rsyslog.strace -ff -D /usr/sbin/rsyslogd -n -iNONE` |
| logger | `strace -yy --write=all -e trace='/(read|write|send|recv).*' -f logger hello` |
