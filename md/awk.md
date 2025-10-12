---
title: "AWK"
---

### Field
```sh
  
# $0: entire line.
$ echo 'one two three' | awk '{print $0}'

# $1: first field, $2: second field,
$ echo 'one two three' | awk '{print $1, $2}'
  
```


### Separator
```sh
  
# default: whitespace

# split on colon
$ awk -F ':'
  
# split on colon or comma
$ awk -F '[:,]'

# split on a regex
$ awk -F 'magiccode*'
  
```


### Printf
```yaml
    
printf format, ...
format: %[flags][width][.precision][length]specifier
    flags:      -, +, #, 0, (space).
    width:      *, (number).
    precision:  *, (number).
    length:     hh, h, ll, l, j, z, t, L, (none).
    specifier:  d, i, u, o, x, X, f, F, e, E, g, G, a, A, c, s, p, n, %.
  
```

```sh
  
# padding space to fixed length 8 
$ echo "15" | awk '{printf "%8d", $1}'
      15

# padding zero to fixed length 8
$ echo "15" | awk '{printf "%08d", $1}'
00000015

# padding zero to fixed length 8 in hex
$ echo "15" | awk '{printf "%08x", $1}'
0000000f
$ echo "15" | awk '{printf "%#08x", $1}'
0x00000f
  
```


### Selection
```sh
  
syntax: awk 'pattern {command}'

# select by comparision
$ echo 'Age 20' | awk '$2 >= 18 {print $0 " adult"}'
$ echo 'Bob 20' | awk '$1 == "Bob" {print $0}'

# select by finding text in line
$ echo 'Bob 20' | awk '/Bob/ {print $0}'
  
```


### Functions
- sub(regex, replace, [from ])
```sh
  
# Replace colon (:) by empty string
$ echo 'Bob: 20' | awk 'sub(":", "") {print}'
  
```