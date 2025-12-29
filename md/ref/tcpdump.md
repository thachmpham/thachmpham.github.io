---
title: tcpdump
---

:::::::::::::: {.columns}
::: {.column}

| Input Output | |
|:-------------|-------------|
| `-D` | List interfaces |
| `-i` | Input interface |
| `-r` | Read from file |
| `-r` | Read from file |
| `-w` | Write to file |
| `-c 5` | Exit after 5 packets |

:::
::: {.column}

| Display |  |
|:-------------|-------------|
| `-xx` | Hex only |
| `-XX` | Hex + ascii |
| `-#` | Display frame number |
| `-e` | Display link header |

:::
::: {.column}

| Time |  |
|:-------------|-------------|
| `-ttt` | delta between frames |
| `-tttt` | date |
| `-ttttt` | delta since frame 1 |

:::
::::::::::::::


:::::::::::::: {.columns}
::: {.column width=20%}

| stdio |  |
|:-------------|-------------|
| `-l` | Flush on end of line  |
| `-U` | flush on end of packet |

:::
::: {.column width=80%}

| grep & awk |
|:-----------|
| `tcpdump -i interface -l | grep --line-buffered expr` |
| `tcpdump -i interface -l | grep --line-buffered expr | awk '{print $3}'` |
| `tcpdump -i interface -l | grep --line-buffered expr | awk '{print $3; fflush()}' | python3 decode.py` |


:::
::::::::::::::


