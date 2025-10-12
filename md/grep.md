---
title: GREP
---

### Perl Regex
```sh
  
grep -P
grep --perl-regexp
  
```

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


### Commands
:::::::::::::: {.columns}

::: {.column}
| | Control |
|-------------|-------------|
| `-i` | Ignore case|
| `-w` | Whole word |
| `-l` | List files |
| `-h` | No file name |
| `-A N` | N lines after |
| `-B N` | N lines before |
| `-C N` | N lines before, after |
| `--line-buffered` | |
:::


::: {.column}
| File Selection |
|-------------|
| `--include="*.hpp" --include="*.cpp"`|
| `--exclude="*.log" --exclude="*.md"`|
| `--exclude-dir="src"` |
:::

::::::::::::::

### References
- [Perl Quick Reference](https://perldoc.perl.org/perlrequick)
- [Perl Reference](https://perldoc.perl.org/perlre)
