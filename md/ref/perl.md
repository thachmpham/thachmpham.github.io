---
title: PERL
---


:::::::::::::: {.columns}
::: {.column width=30%}

## Commands

| | Input Control |
|-------------|-------------|
| `-e` | Execute command |
| `-E` | Execute with features |

<br>

| | Output Control |
|-------------|-------------|
| `-p` | Print all lines |
| `-n` | Not print automatically |

:::
::: {.column width=30%}

## Split

| Pattern | |
|-------------|-------------|
| `-F<regex>` | Split on regex |
| `-F:` | Split on `:` |
| `-F'[:,]'` | Split on `:` or `,` |

<br>

| Fields |  |
|-------------|-------------|
| `$F[0] $F[1]` | Fields 0, 1 |
| `@F` | All fields |

Split on `:`. Print fields 0, 1.  
`-F: -e 'print "$F[0] $F[1]\n"'`

Split on `:`. Print all fields.  
`-F: -e 'print "@F\n"'`


:::
::: {.column width=30%}

## Subtitude 
| Match & Replace | |
|-------------|-------------|
| `-pe 's/foo/bar/'` | Replace first `foo` by `bar` |
| `-pe 's/foo/bar/g'` | Replace all `foo` by `bar` |
| `-pe 's/foo/bar/i'` | Ignore case insensitive |
| `-pe 's/(foo) (bar)/$2 $1/'` | Capure groups. Swap `foo`, `bar` |

:::
::::::::::::::


:::::::::::::: {.columns}
::: {.column}

## Groups 

| Extract Matches | |
|-------------|-------------|
| `-ne 'print "$1\n" if /name=(\w+)/'` | Extract 1st match |
| `'print "$1 $2\n" if /name=(\w+) age=(\d+)/'` | Extract 1st, 2nd |

:::
::: {.column}

## Look 

| | | Lookahead | |
|-------------|-------------|-------------|-------------|
| Positive | `car (?=run)` | Match `car` only if followed by `run` | `car run` |
| Negative | `car (?!run)` | Match `car` only if not followed by `run` | `car stop` |

<br>

| | | Lookbehind | |
|-------------|-------------|-------------|-------------|
| Positive | `(?<=red) car` | Match `car` only if preceded by `red` | `red car` |
| Negative | `(?<!red) car` | Match `cat` only if not preceded by `red` | `blue car` |

:::
::::::::::::::

## Wildcards

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

:::
::: {.column}

| | Boundary |
|-------------|-------------|
| `hello` | `.*hello.*`  |
| `\bhello\b` | `hello`  |
| `\Bhello\B` | `\whello\w` |

<br>

| | Group |
|-------------|-------------|
| `(a|b)c` | Match ab, ac |

:::
::::::::::::::
