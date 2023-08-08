---
weight: 1
bookFlatSection: false
title: "MapReduce"
---

# Add unit test for wc.go

```zsh
vi src/mrapps/ut_wc.go
```


```go
// file: src/mrapps/ut_wc.go
package main

import "fmt"

func main() {
    {
        out := Map("input.txt", "hello world and hello go")
        fmt.Println("Map output: ", out)
    }

    {
        out := Reduce("hello", [] string {"1", "1"})
        fmt.Println("Reduce output:", out)
    }
}
```


```zsh
go run mrapps/wc.go mrapps/ut_wc.go
```
Map output:  [{hello 1} {world 1} {and 1} {hello 1} {go 1}]
Reduce output: 2
