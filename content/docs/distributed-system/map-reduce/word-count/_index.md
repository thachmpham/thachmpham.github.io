---
weight: 1
bookFlatSection: false
title: "Distributed Word Count"
bookToc: true
---

# Distributed Word Count
During this lab, we will construct a MapReduce framework. Our task involves:
- Implement a worker process that calls application Map and Reduce functions and handles reading and writing files
- Implement a coordinator process that assigns tasks to workers and manages situations involving worker failures.  

Clone source code and init.
```zsh
git clone git://g.csail.mit.edu/6.5840-golabs-2023 6.5840
cd 6.5840
Makefile src
```

## 1. Word Counting
`src/mrapps/wc.go` consists of two functions: Map and Reduce. `wc` stands for word count. These functions are used to count occurences of words in files later.

## 1.1. Map
### Declaration
```go
func Map(filename string, contents string) []mr.KeyValue
```
`filename`: the name of the input file.  
`contents`: file content.

### Description
- Divides the `contents` into words using a space as the separator.  
- Return a list of maps where each map has a word as the key and a value of 1.

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
- `key`: word.  
- `values`: a list of string "1".

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

## 2. Sequential
Prior to delving into the MapReduce solution, let's begin by exploring the sequential approach to accomplishing this task. The file `src/main/mrsequential.go` encompasses the sequential algorithm used to tally word occurrences in files. We'll outline the steps of the algorithm below, accompanied by sample input and output for each phase.

**Recommendation:** Before proceeding further, it's advisable to review the `main()` function within `src/main/mrsequential.go`.

- Read each input file, pass it to Map, accumulate the `intermediate` map output.
```text
input_file = hello.txt
content = "hello world and hello go"

intermediate = [{hello 1} {world 1} {and 1} {hello 1} {go 1}]
```

- Sort `intermediate`
```text
intermediate = [{and 1} {go 1} {hello 1} {hello 1} {world 1}]
```

- Aggregate the values in the `intermediate` based on their keys.
```text
intermediate = [{and 1} {go 1} {hello 1} {hello 1} {world 1}]

|   key   | values      |
|:-------:|:-----------:|
|   and   |     [1]     |
|   go    |     [1]     |
|  hello  |   [1, 1]    |
|  world  |     [1]     |
```

- Call `Reduce` on each distinct key, values.
```text
|   key   | values      | reduce output |
|:-------:|:-----------:|:-------------:|
|   and   |     [1]     |       1       |
|   go    |     [1]     |       1       |
|  hello  |   [1, 1]    |       2       |
|  world  |     [1]     |       1       |
```
- The output will be saved in file `mr-out-0`.

## 2.1. Testcase 1
### Description
- Create a text file named `hello.txt`.  
- Run `mrsequential.go` to count of words.

### Run
```zsh
echo 'hello world and hello go' > hello.txt

cd src/main
go build -buildmode=plugin ../mrapps/wc.go

go run mrsequential.go wc.so hello.txt
```

The output is saved in the file named `mr-out-0`.
```text
and 1
go 1
hello 2
world 1
```

## 2.2. Testcase 2
### Description
Count the occurences of words in all files having name pattern `pg*.txt`, encompasses the following files:
- `pg-being_ernest.txt`
- `pg-dorian_gray.txt`
- `pg-frankenstein.txt`
- `pg-grimm.txt`
- `pg-huckleberry_finn.txt`
- `pg-metamorphosis.txt`
- `pg-sherlock_holmes.txt`
- `pg-tom_sawyer.txt`

### Run
```zsh
go run mrsequential.go wc.so pg*.txt
```

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

## 3. MapReduce
MapReduce is a programming model and computational framework designed for processing and generating large-scale data sets in a parallel and distributed manner. It was popularized by Google and has become a fundamental concept in big data processing. The MapReduce framework is used for processing tasks such as data transformation, filtering, sorting, and aggregation.  
  
The MapReduce model consists of two main phases: the Map phase and the Reduce phase.  
  
**Map Phase:**  
- In this phase, the input data is divided into smaller chunks, and a user-defined function called the "map function" is applied to each chunk independently and in parallel.
- The map function takes an input key-value pair and produces intermediate key-value pairs as output.

**Shuffle and Sort Phase:**  
- After the map phase, the intermediate key-value pairs are grouped and shuffled based on their keys. This step ensures that all intermediate values associated with the same key are brought together in one place.
- The intermediate key-value pairs are then sorted by key to prepare them for the next phase.

**Reduce Phase:**  
- In this phase, another user-defined function called the "reduce function" is applied to the sorted intermediate key-value pairs.
- The reduce function takes a key and its associated list of values and produces one or more output key-value pairs.

## 3.1. Assigment
Our job is to implement a distributed MapReduce, consisting of two programs, the coordinator and the worker.  

{{< columns >}}
## Coordinator
- There will be just one coordinator process.
- Its role is to listen and assign tasks to workers.
- The coordinator should notice if a worker has not completed its task in a reasonable amount of time and give the same task to a different worker.

<---> 
## Workers
- There will be one or more worker processes executing in parallel.
- Each worker process will ask the coordinator for a task.
- The task might involve reading input from one or multiple files, performing computations, and then writing the output of the task to one or several files.

{{< /columns >}}

