---
title: "GBD: Break"
---

:::::::::::::: {.columns}
::: {.column}

|           | Linespec Location |
|:----------|:----------|
|Line       | `break 10`|
|           | `break hello.c::10`|
|Offset     | `break -3` |
|           | `break +3` |
|Function   | `break sum`|
|           | `break hello.c::sum` |
|           | `break A::sum` |

:::
::: {.column}

|           | Explicit Location       |             |
|:----------|:------------------------|:------------|
| -source   | `break -source hello.c sum`| Function sum in hello.c |
| -qualified| `break -qualified sum `    | Match sum, not A::sum |

:::
::: {.column}

|           | Address Location        |             |
|:----------|:------------------------|:------------|
|           |`break *sum+16`          | Instruction 16 in sum |
|           |`break 0x`0x400670       | Instruction at 0x400670 |

:::
::::::::::::::