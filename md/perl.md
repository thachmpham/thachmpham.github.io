---
title: PERL
---


### Split
:::::::::::::: {.columns}
::: {.column}

| Pattern | |
|-------------|-------------|
| `-F<regex>` | Split on regex |
| `-F:` | Split on `:` |
| `-F'[:,]'` | Split on `:` or `,` |

:::
::: {.column}

| Fields |  |
|-------------|-------------|
| `$F[0] $F[1]` | Fields 0, 1 |
| `@F` | All fields |

:::
::::::::::::::

| | |
|-------------|-------------|
| `-F: -e 'print "$F[0] $F[1]\n"'` | Split on `:`. Print fields 0, 1. |
| `-F: -e 'print "@F\n"'` | Split on `:`. Print all fields. |

<br>

### Subtitude 
| Matching | |
|-------------|-------------|
| `-pe 's/foo/bar/'` | Replace first `foo` by `bar` |
| `-pe 's/foo/bar/g'` | Replace all `foo` by `bar` |
| `-pe 's/foo/bar/i'` | Ignore case insensitive |

| Capture Groups | |
|-------------|-------------|
| `-pe 's/(foo) (bar)/$2 $1/'` | Capure groups. Swap `foo`, `bar` |

<br>

### Options
:::::::::::::: {.columns}
::: {.column}

| | Input Control |
|-------------|-------------|
| `-e` | Execute command |
| `-E` | Execute with features |

:::
::: {.column}

| | Output Control |
|-------------|-------------|
| `-p` | Print all lines |
| `-n` | Not print automatically |

:::
::::::::::::::

<br>

### Regex
:::::::::::::: {.columns}
::: {.column}

| | Class |
|-------------|-------------|
| `.` | Any |
| `\d` | Digit |
| `\D` | Negated of digit |
| `\w` | Word `[0-9a-zA-Z_]` |
| `\W` | Negated of word |
| `\s` | Whitespace |
| `\s` | Negated of whitespace |
| `[abc]`| Character class [a,b,c] |
| `^[abc]` | Negated of [a,b,c]|
| `^` | Begin of line |
| `$` | End of line |
| `\` | Escape next character |

<br>

| | Group |
|-------------|-------------|
| `(a|b)c` | Match ab, ac |

:::
::: {.column}

| | Quantity |
|-------------|-------------|
| `*` | 0-N |
| `?` | 0-1 |
| `+` | 1-N |
| `a{2}` | aa |
| `a{2,4}` | aa - aaaa |

<br>

| | Greedy |
|-------------|-------------|
| `a*b` |  Longest a*b |
| `a*?b` | Shortest a*b |

<br>

| | Boundary |
|-------------|-------------|
| `hello` | `.*hello.*`  |
| `\bhello\b` | `hello`  |
| `\Bhello\B` | `\whello\w` |

:::
::::::::::::::

| | Look ahead | |
|-------------|-------------|-------------|
| Positive | `(?=hello)` | Next is hello |
| Negative | `(?!hello)` | Next is not hello |

