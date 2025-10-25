---
title: GREP
---

### Perl Regex
```sh
  
grep -P
  
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

| | Look ahead | |
|-------------|-------------|-------------|
| Positive | `(?=hello)` | Next is hello |
| Negative | `(?!hello)` | Next is not hello |


### Commands
:::::::::::::: {.columns}

::: {.column}

| | Matching Control |
|-------------|-------------|
| `-i` | Ignore case|
| `-w` | Whole word |
| `-o` | Print only matched parts |
| `-m` | Max count |
| `-a` | Treat binary as text |
:::

::: {.column}
| | Output Control |
|-------------|-------------|
| `-l` | List files |
| `-h` | No file name |
| `-o` | Print only matched parts |
| `-A N` | N lines after |
| `-B N` | N lines before |
| `-C N` | N lines before, after |
| `--line-buffered` | Line buffering on output |
:::

::::::::::::::


| File Selection |
|-------------|
| `--include="*.hpp" --include="*.cpp"`|
| `--exclude="*.log" --exclude="*.md"`|
| `--exclude-dir="src"` |

