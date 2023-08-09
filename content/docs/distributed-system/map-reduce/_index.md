---
weight: 1
bookFlatSection: false
title: "Lab 1: MapReduce"
bookToc: true
---

# Lab 1: MapReduce
## 1. `src/mrapps/wc.go`
`src/mrapps/wc.go` consists of two functions: Map and Reduce.  
`wc` stands for word count.

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
### 2.1. Testcase 1
Build `wc.so` library.
```zsh
cd src/main
go build -buildmode=plugin ../mrapps/wc.go
```

Create a input file.
```zsh
echo 'hello world and hello go' > hello.txt
```

Run word counting using the sequential algorithm. `wc.so` is the library which has Map and Reduce functions. `hello.txt` is the input text file that we want to count words.
```zsh
go run mrsequential.go wc.so hello.txt
```

The output is saved in the file named `mr-out-0`.
```text
and 1
go 1
hello 2
world 1
```

### 2.2. Testcase 2
```zsh
go run mrsequential.go wc.so pg*.txt
```
`pg*.txt`: the pattern for input files, encompasses the following files:
- `pg-being_ernest.txt`
- `pg-dorian_gray.txt`
- `pg-frankenstein.txt`
- `pg-grimm.txt`
- `pg-huckleberry_finn.txt`
- `pg-metamorphosis.txt`
- `pg-sherlock_holmes.txt`
- `pg-tom_sawyer.txt`

The output is saved in the file named `mr-out-0`.
```text
A 509
ABOUT 2
ACT 8
ACTRESS 1
ACTUAL 8
ADLER 1
ADVENTURE 12
ADVENTURES 7
AFTER 2
AGREE 16
AGREEMENT 8
...
```
