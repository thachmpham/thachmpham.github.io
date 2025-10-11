---
title: tcpdump
---


### IO
- List interfaces.
```sh
  
tcpdump --list-interfaces
tcpdump -D
  
```

- Capture.
```sh
  
tcpdump -i interface
tcpdump -i interface -w outfile
  
```

- Read from file.
```sh
  
tcpdump -r infile

# read only N packets
tcpdump -r infile -c N 
  
```


### Output Format
- Hex and ascii.
```sh
  
# hex only
-xx

# hex and ascii
-XX
  
```

- Frame number
```sh
  
--number
-#
  
```

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

- Filter packets containing specific bytes at a given offset.
```sh
  
'ether[0x30] == 0x63'
'ether[0x30:2] == 0x6368'
  
```


