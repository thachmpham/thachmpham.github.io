---
title: tcpdump
---


# Input & Output

:::::::::::::: {.columns}
::: {.column width=30%}

List interfaces.
```sh
-D
```

Capture.
```sh
-i interface
-i interface 'expr_string'
-i interface -F expr_file
```

Read from file.
```sh
-r infile
-r infile -c N	# only N packets
```

:::
::: {.column width=70%}


Write to file.
```sh
-w filename
```

stdout.
```sh 
-l	# flush on end of line
-U	# flush on end of packet
```

grep & awk.
```sh
tcpdump -i interface -l | grep --line-buffered expr
tcpdump -i interface -l | grep --line-buffered expr | awk '{print $3}'
tcpdump -i interface -l | grep --line-buffered expr | awk '{print $3; fflush()}' | python3 decode.py
```

:::
::::::::::::::


# Display Format
:::::::::::::: {.columns}
::: {.column width=60%}

Hex and ascii.
```sh
-xx	# hex only
-XX	# hex and ascii
```

Display link layer header.
```sh
-e
```

:::
::: {.column}

Time.
```sh
-ttt    # delta between frames
-tttt   # date
-ttttt  # delta since frame 1     
```

Display frame number.
```sh
--number
```

:::
::::::::::::::
