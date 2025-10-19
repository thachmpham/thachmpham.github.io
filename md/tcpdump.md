---
title: tcpdump
---


### IO
:::::::::::::: {.columns}

::: {.column}
- List interfaces.
```sh
  
-D
  
```

- Capture.
```sh
  
-i interface
-i interface -w outfile
  
```
:::


::: {.column}
- Read from file.
```sh
  
-r infile

# read only N packets
-r infile -c N 
  
```

:::

::::::::::::::

### Output Format
:::::::::::::: {.columns}

::: {.column}
- Hex and ascii.
```sh
  
# hex only
-xx

# hex and ascii
-XX
  
```

- Frame number.
```sh
  
--number
-#
  
```

:::


::: {.column}
- Link-level header.
```sh
  
-e
  
```


- Time.
```sh
  
-ttt    # delta between frames
-tttt   # date
-ttttt  # delta since frame 1     
  
```

:::

::::::::::::::

### Redirect Output
- Redirect to stdout.
```sh
  
# flush on end of line  
-l

# flush on end of packet
- U
  
```

- grep & awk.
```sh
  
tcpdump -i interface -l | grep --line-buffered expr
tcpdump -i interface -l | grep --line-buffered expr | awk '{print $3}'
tcpdump -i interface -l | grep --line-buffered expr | awk '{print $3; fflush()}' | python3 decode.py
  
```


### Filter Expression
- Expression syntax.
```sh
  
man pcap-filter
  
```

- Use expression.
```sh
  
tcpdump -r infile 'expr_string'
tcpdump -r infile -F expr_file
  
```

:::::::::::::: {.columns}

::: {.column}
- Offset.
```sh
  
'ether[0x30] == 0x63'
'ether[0x30:2] == 0x6368'
  
```
:::


::: {.column}
- Length.
```sh
  
'len == 58'
'len != 58'
  
```
:::

::::::::::::::
