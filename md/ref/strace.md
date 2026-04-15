---
title: "strace"
---



:::::::::::::: {.columns}
::: {.column}

| Filter |         |        |
|:-------|:--------|:-------|
| `syscalls`    | `-e trace=read`                   | Single syscall    |
|                 | `-e trace=read,write`             | Multiple syscalls |
|                 | `-e trace='!read'`                | Not syscall  |
| `syscall regex` | `-e trace='/open.*'`              | Regex        |
| `syscall sets`  | `-e trace=file`                   | File callset |
|                 | `-e trace=file -e trace='!write'` | File callset |
| `fds`           | `-e fds=1`   | Single fd |
|                 | `-e fds=1,2` | Multiple fds |
|                 | `-e fds=!1`  | Not fd |

:::
::: {.column}

| Filter          |              |
|:----------------|:-------------|
| status          | `-e status=successful` |
|                 | `-e status=failed`     |
| signal          | `-e signals=SIGIO`     |
| path            | `-P hello`             |

| Fork            |                                    |
|:----------------|:-----------------------------------|
| `-p pid`        | `--attach=pid`                        |
| `-o myfile`     | Output to file                        |
| `-o myfile -f`  | Follow child, output to file          |
| `-o myfile -ff` | Each child output to a separated file |
| `-o myfile -D`  | Detached mode                         |

:::
::::::::::::::


:::::::::::::: {.columns}
::: {.column}

| Decode    |               |                                        |
|:----------|:------------- |:---------------------------------------|
| string    | `--write=1`   | On writes to fd 1, show data in hex + ascii    |
|           | `--write=all` | On writes to all fds, show data in hex + ascii |
|           | `--read=0`    | On reads from fd 0, show hex + ascii    |
|           | `--read=all`  | On reads from all fds, show hex + ascii |
|           | `-x`          | Print non-ascii strings in hex  |
|           | `-xx`         | Print all strings in hex        |
|           | `-x -s 32`    | Non-ascii strings in hex, max length 32 |


:::
::: {.column}

| Decode    |               |                                        |
|:----------|:------------- |:---------------------------------------|
| fds       | `-y`          | Show file path associated with fds      |
|           | `-yy`         | Show all info associated with fds       |
| backtrace |`-k`           | `--stack-trace=symbol` |
|           |`-kk`          | `--stack-trace=source` |

:::
::::::::::::::

* * * * *

### Real Cases

| Service      | Patch                                                                        |
|:-------------|:-----------------------------------------------------------------------------|
| rsyslog | `ExecStart=strace -yy -o /tmp/rsyslog.strace -ff -D /usr/sbin/rsyslogd -n -iNONE` |
| logger | `strace -yy --write=all -e trace='/(read|write|send|recv).*' -f logger hello`      |
| tipc   | `strace -yy -x --read=all --write=all -f -p $(pidof hello_server)`                 |
