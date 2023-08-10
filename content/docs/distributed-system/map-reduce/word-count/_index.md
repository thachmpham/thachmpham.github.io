---
weight: 1
bookFlatSection: false
title: "Distributed Word Count"
bookToc: true
---

# Distributed Word Count
## 1. Introduction
Distributed Word Count is a classic example often used to illustrate the MapReduce programming model for processing and analyzing large datasets. In this scenario, the goal is to count the occurrences of each word in a given collection of text documents across a distributed cluster of computers. The MapReduce approach divides the task into two main phases: the Map phase and the Reduce phase.

While the example is a simplified illustration, it forms the basis for more complex data processing tasks that can be accomplished using the MapReduce paradigm. It showcases the elegance of the model in solving real-world challenges in data analysis and processing on distributed computing environments.

In this blog post, we will utilize **Golang** to develop a word counting program through two different approaches: a **Sequential** solution and a **MapReduce** solution.


## 2. Sequential Solution
## 2.1. Description
Before delving into the MapReduce solution, let's begin by exploring the sequential approach to accomplishing this task. We'll outline the steps of the algorithm below, accompanied by sample input and output for each phase.

**Map Phase:**
- Input is a set of text files. For each file, a Map function is applied. The Map function tokenizes the text, separates it into words, and emits key-value pairs. The key is the word, and the value is a count of 1 for each occurrence of the word.
```text
input_file = [hello.txt, hi.txt]

content_hello = "hello world and hello go"      // content of hello.txt
content_hi = "hi go"                            // content of hi.txt

Map(hello.txt, content_hello)   => [{hello 1} {world 1} {and 1} {hello 1} {go 1}]
Map(hi.txt, content_hi)         => [{hi 1} {go 1}]

intermediate = [{hello 1} {world 1} {and 1} {hello 1} {go 1} {hi 1} {go 1}]
// intermediate is the list of all previous Map outputs
```

**Shuffle and Sort:**
- Intermediate Data: The emitted key-value pairs from all Map are sorted and grouped based on their keys. All occurrences of a specific word are brought together.

```text
// sort intermediate
intermediate = [{and 1} {go 1} {go 1} {hello 1} {hello 1} {hi 1} {world 1}]

// occurrences of a word are brought together
|   key   | values      |
|:-------:|:-----------:|
|   and   |   [1]       |
|   go    |   [1, 1]    |
|  hello  |   [1, 1]    |
|   hi    |   [1]       |
|  world  |   [1]       |
```

**Reduce Phase:**
- Reduce Function: For each unique word key, a Reduce function receives the list of counts associated with that word. The Reduce function sums up these counts to compute the total occurrences of the word.
```text
Reduce("and", [1])      => 1
Reduce("go", [1, 1])    => 2
Reduce("hello", [1, 1]) => 2
Reduce("hi", [1])       => 1
Reduce("world", [1])    => 1

Summary:
|   key   | values      | reduce output |
|:-------:|:-----------:|:-------------:|
|   and   |     [1]     |       1       |
|   go    |   [1, 1]    |       2       |
|  hello  |   [1, 1]    |       2       |
|   hi    |     [1]     |       1       |
|  world  |     [1]     |       1       |
```

**Result:**
- Output: The reducer emits the word as the key and its total count as the value.
```text
|   key   |    output     |
|:-------:|:-------------:|
|   and   |       1       |
|   go    |       2       |
|  hello  |       2       |
|  world  |       1       |
```

## 2.2. Implementation
### `wc.go`
File `wc.go` provides Map and Reduce functions which support for word counting.

```go
package main

import "unicode"
import "strings"
import "strconv"

type KeyValue struct {
	Key   string
	Value string
}

// for sorting by key.
type ByKey []KeyValue

// for sorting by key.
func (a ByKey) Len() int           { return len(a) }
func (a ByKey) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByKey) Less(i, j int) bool { return a[i].Key < a[j].Key }

/*
	The Map function tokenizes the text, separates it into words, 
	and emits key-value pairs.The key is the word, 
	and the value is a count of 1 for each occurrence of the word.

	Samples:
	Map("hello.txt", "hello world and hello go")
 	==> [{hello 1} {world 1} {and 1} {hello 1} {go 1}]

 	Map("hi.txt", "hello go")
 	==> [{hello 1} {go 1}]
*/
func Map(filename string, contents string) [] KeyValue {
	// function to detect word separators.
	ff := func(r rune) bool { return !unicode.IsLetter(r) }

	// split contents into an array of words.
	words := strings.FieldsFunc(contents, ff)

	kva := [] KeyValue{}
	for _, w := range words {
		kv := KeyValue{w, "1"}
		kva = append(kva, kv)
	}
	return kva
}


/*
	Reduce function receives the list of counts associated with key.
	The Reduce function sums up these counts to compute the total occurrences of the word.

	Samples:
	Reduce("hello", []string{"1", "1"})		=> 2
	Reduce("hi", ["1"])       => 1
*/
func Reduce(key string, counts []string) string {
	// return the number of occurrences of this word.
	return strconv.Itoa(len(counts))
}
```

### `sequential.go`
In file `sequential.go`, we implement sequential logic to to compute word occurrences within files.
```go
package main

import "fmt"
import "os"
import "io/ioutil"
import "sort"

func main() {
	if len(os.Args) < 2 {
		fmt.Fprintf(os.Stderr, "Usage: mrsequential <filename_1> [filename_2]...\n")
		os.Exit(1)
	}

	fmt.Println("Phase Map")

	intermediate := [] KeyValue{}
	for _, filename := range os.Args[1:] {
		file, _ := os.Open(filename)
		content, _ := ioutil.ReadAll(file)
		file.Close()
		
		kva := Map(filename, string(content))
		fmt.Println("Map", filename, "=>", kva)

		intermediate = append(intermediate, kva...)
		fmt.Println("intermediate:", intermediate)
	}

	fmt.Println("\nPhase Suffle and Sort")

	sort.Sort(ByKey(intermediate))
	fmt.Println("sorted intermediate:", intermediate)

	oname := "output.txt"
	ofile, _ := os.Create(oname)

	fmt.Println("\nPhase Reduce")
	i := 0
	for i < len(intermediate) {
		j := i + 1
		for j < len(intermediate) && intermediate[j].Key == intermediate[i].Key {
			j++
		}
		values := []string{}
		for k := i; k < j; k++ {
			values = append(values, intermediate[k].Value)
		}

		count := Reduce(intermediate[i].Key, values)
		fmt.Println("Reduce", intermediate[i].Key, "=>", count)

		// Save reduce result to output file
		fmt.Fprintf(ofile, "%v %v\n", intermediate[i].Key, count)

		i = j
	}

	ofile.Close()
	fmt.Println("\nOutput saved to output.txt")
}
```

## 2.3. Testcase
In the following test scenario, we'll generate two input files: hello.txt and hi.txt. Subsequently, we will execute the sequential.go program to compute word occurrences.

```zsh
mkdir input
echo "hello world and hello go" > input/hello.txt
echo "hi go" > input/hi.txt

go run wc.go sequential.go input/hello.txt input/hi.txt
```

Output is saved in output.txt.
```text
and 1
go 2
hello 2
hi 1
world 1
```

You can check log in the terminal for more detailed information.
```text
Phase Map
Map input/hello.txt => [{hello 1} {world 1} {and 1} {hello 1} {go 1}]
intermediate: [{hello 1} {world 1} {and 1} {hello 1} {go 1}]
Map input/hi.txt => [{hi 1} {go 1}]
intermediate: [{hello 1} {world 1} {and 1} {hello 1} {go 1} {hi 1} {go 1}]

Phase Suffle and Sort
sorted intermediate: [{and 1} {go 1} {go 1} {hello 1} {hello 1} {hi 1} {world 1}]

Phase Reduce
Reduce and => 1
Reduce go => 2
Reduce hello => 2
Reduce hi => 1
Reduce world => 1

Output saved to output.txt
```
