---
title: PERL
---

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


### Options
:::::::::::::: {.columns}

::: {.column}

| | |
|-------------|-------------|
| `-e` | Execute command |
| `-E` | Execute with features |

:::

::: {.column}
| | |
|-------------|-------------|
| `-p` | Print all lines |
| `-n` | Not print automatically |

:::

::::::::::::::


### Split
| | |
|-------------|-------------|
| `-F<pattern>` | Split on regex pattern |
| `-F:` | Split on `:` |
| `-F'[:,]'` | Split on `:` or `,` |
| `$F[0] $F[1]` | Fields 0, 1 |
| `@F` | All fields |
| | |
| `-F: -e 'print "$F[0] $F[1]\n"'` | Split on `:`. Print fields 0, 1. |
| `-F: -e 'print "@F\n"'` | Split on `:`. Print all fields. |


### Subtitude 
| | |
|-------------|-------------|
| `-pe 's/foo/bar/'` | Replace first `foo` by `bar` |
| `-pe 's/foo/bar/g'` | Replace all `foo` by `bar` |
| `-pe 's/foo/bar/i'` | Ignore case insensitive |
| `-pe 's/(foo) (bar)/$2 $1/'` | Capure groups. Swap `foo`, `bar` |


### References
- [Perl Quick Reference](https://perldoc.perl.org/perlrequick)
- [Perl Reference](https://perldoc.perl.org/perlre)
