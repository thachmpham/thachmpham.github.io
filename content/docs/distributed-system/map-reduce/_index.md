---
weight: 1
bookFlatSection: false
title: "Lab 1: MapReduce"
bookToc: true
---

# Lab 1: MapReduce
## 1. `src/mrapps/wc.go`
`src/mrapps/wc.go` consists of two functions: Map and Reduce.
## 1.1. Map
### Declaration
```go
func Map(filename string, contents string) []mr.KeyValue
```
`filename`: the name of the input file.  
`contents`: file content.

### Description
The function divides the `contents` into words using a space as the separator.  
Generate a list of dictionaries where each dictionary has a word as the key and a value of 1.

### Sample code
```go
package main

import "fmt"

func main() {
    out := Map("input.txt", "hello world and hello go")
    fmt.Println("Map output: ", out)

    // output:
    // [{hello 1} {world 1} {and 1} {hello 1} {go 1}]
}
```

## 1.2. Reduce
### Declaration
```go
func Reduce(key string, values []string) string
```
`key`: word.  
`values`: a list of string "1".

### Description
Return the number of elements in list `values`.

### Sample code
```go
package main

import "fmt"

func main() {
    out := Reduce("hello", [] string {"1", "1"})
    fmt.Println("Reduce output:", out)

    // output: 2
}
```

## 2. `src/main/mrsequential.go`
### Run
```zsh
cd src/main
go build -buildmode=plugin ../mrapps/wc.go
rm mr-out*
go run mrsequential.go wc.so pg*.txt
```