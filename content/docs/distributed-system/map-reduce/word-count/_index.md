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
## 2.1 Description
Before delving into the MapReduce solution, let's begin by exploring the sequential approach to accomplishing this task. 

In the below example, we will **count occurences of words** in two files `hello.txt` and `hi.txt`.  

Content of file `hello.txt`.
```text
hello world and hello go
```

Content of file `hi.txt`.
```text
hi go
```

And we expect the output will be like that.
```text
and 1
go 2
hello 2
hi 1
world 1
```

Now, we will now explore the functioning of each phase and analyze the input and output of each one.  

### 2.1.1. Map Phase
- Input is a set of text files. For each file, a Map function is applied. The Map function tokenizes the text, separates it into words, and emits key-value pairs. The key is the word, and the value is a count of 1 for each occurrence of the word.
```text
// Call Map function on hello.txt

              hello world and hello go
                          |
                          v
                    Map function
                          |
                          v
  {hello 1} {world 1} {and 1} {hello 1} {go 1}

// Call Map function on hi.txt

                      hi go
                        |
                        v
                    Map function
                        |
                        v
                  {hi 1} {go 1}

// Sum up the results
intermediate = {hello 1} {world 1} {and 1} {hello 1} {go 1} {hi 1} {go 1}
```

### 2.1.2. Shuffle & Sort Phase
- Intermediate Data: The emitted key-value pairs from all Map are sorted and grouped based on their keys. All occurrences of a specific word are brought together.

```text
{hello 1} {world 1} {and 1} {hello 1} {go 1} {hi 1} {go 1}
                        |
                        v
                Sort intermediate by key
                        |
                        v
{and 1} {go 1} {go 1} {hello 1} {hello 1} {hi 1} {world 1}
                        |
                        v
                    Group by key
                        |
                        v
            _________________________
            |   key   | values      |
            |:-------:|:-----------:|
            |   and   |   [1]       |
            |   go    |   [1, 1]    |
            |  hello  |   [1, 1]    |
            |   hi    |   [1]       |
            |  world  |   [1]       |

```

### 2.1.3. Reduce Phase
- Reduce Function: For each unique word key, a Reduce function receives the list of counts associated with that word. The Reduce function sums up these counts to compute the total occurrences of the word.
```text
// Reduce function returns the number of elements in the values list.

|   key   | values      |     Reduce    |
|:-------:|:-----------:|:-------------:|
|   and   |     [1]     |       1       |
|   go    |   [1, 1]    |       2       |
|  hello  |   [1, 1]    |       2       |
|   hi    |     [1]     |       1       |
|  world  |     [1]     |       1       |
```

### 2.1.4. Result
- Output: The reducer emits the word as the key and its total count as the value.
```text
|   key   |  occurrence   |
|:-------:|:-------------:|
|   and   |       1       |
|   go    |       2       |
|  hello  |       2       |
|  world  |       1       |
```

## 2.2. Implementation
### 2.2.1. `wc.go`
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

### 2.2.2. `sequential.go`
In file `sequential.go`, we implement sequential logic. The program receives a list of input files, compute word occurences, and save to file `output.txt`.

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
### 2.3.1. Prepare Input Data
```zsh
mkdir input
echo "hello world and hello go" > input/hello.txt
echo "hi go" > input/hi.txt
```

### 2.3.2. Run
```zsh
go run wc.go sequential.go input/hello.txt input/hi.txt
```

Output is saved in `output.txt`.
```text
and 1
go 2
hello 2
hi 1
world 1
```

## 2. **MapReduce Solution**
## 2.1. Description
In a MapReduce solution, the Coordinator handles task distribution, progress monitoring, and data management, while Workers execute tasks, process data, and communicate their results back to the Coordinator.

### 2.1.1 Map Phase
**Coordinator**
- Creates map tasks for files: `hello.txt` and `hi.txt`.
- Wait for for task requests from Workers.

```text
     ┌────────┐          ┌────────┐           ┌───────────┐                        
     │Worker_1│          │Worker_2│           │Coordinator│                        
     └───┬────┘          └───┬────┘           └─────┬─────┘                        
         │                   │                      │────┐                         
         │                   │                      │    │ create MapTask hello.txt
         │                   │                      │<───┘                         
         │                   │                      │                              
         │                   │                      │────┐                         
         │                   │                      │    │ create MapTask hi.txt   
         │                   │                      │<───┘                         
         │                   │                      │                              
         │               request task               │                              
         │ ─────────────────────────────────────────>                              
         │                   │                      │                              
         │                   │     request task     │                              
         │                   │ ─────────────────────>                              
         │                   │                      │                              
         │         assign MapTask hello.txt         │                              
         │ <─────────────────────────────────────────                              
         │                   │                      │                              
         │                   │ assign MapTask hi.txt│                              
         │                   │ <─────────────────────                              
     ┌───┴────┐          ┌───┴────┐           ┌─────┴─────┐                        
     │Worker_1│          │Worker_2│           │Coordinator│                        
     └────────┘          └────────┘           └───────────┘        
```

**Worker_1**  
- Involkes Map function on `hello.txt`.
```text
Map("hello world and hello go") => {hello 1} {world 1} {and 1} {hello 1} {go 1}
```
- Generate temporary file names to store the outcomes of the Map function using the format `filename = tmp.<worker id>.<hash>`. Associate keys with their respective files.
```text
| worker | key   | hash | filename    |
|--------|-------|------|-------------|
| 1      | hello | 3    | tmp.1.3  |
| 1      | and   | 6    | tmp.1.6  |
| 1      | go    | 7    | tmp.1.7  |
```

- Write outputs of Map function to corresponding files.
```text
     ┌────────┐           ┌───────┐          ┌───────┐          ┌───────┐
     │Worker_1│           │tmp.1.3│          │tmp.1.6│          │tmp.1.7│
     └───┬────┘           └───┬───┘          └───┬───┘          └───┬───┘
         │ {hello 1} {hello 1}│                  │                  │    
         │ ───────────────────>                  │                  │    
         │                    │                  │                  │    
         │                {and 1}                │                  │    
         │ ──────────────────────────────────────>                  │    
         │                    │                  │                  │    
         │                    │{go 1} {world 1}  │                  │    
         │ ─────────────────────────────────────────────────────────>    
     ┌───┴────┐           ┌───┴───┐          ┌───┴───┐          ┌───┴───┐
     │Worker_1│           │tmp.1.3│          │tmp.1.6│          │tmp.1.7│
     └────────┘           └───────┘          └───────┘          └───────┘
```

**Worker_2**  
- Involkes Map function on `hi.txt`.
```text
Map("hi go") => {hi 1} {go 1}
```

- Generate temporary file names to store the outcomes of the Map function.
```text
| worker | key   | hash | filename    |
|--------|-------|------|-------------|
| 2      | hi    | 2    | tmp.2.2     |
| 2      | go    | 7    | tmp.2.7     |
```

- Write outputs of Map function to corresponding files.
```text
     ┌────────┐          ┌───────┐          ┌───────┐
     │Worker_2│          │tmp.2.2│          │tmp.2.7│
     └───┬────┘          └───┬───┘          └───┬───┘
         │      {hi 1}       │                  │    
         │ ─────────────────>│                  │    
         │                   │                  │    
         │                {go 1}                │    
         │ ────────────────────────────────────>│    
     ┌───┴────┐          ┌───┴───┐          ┌───┴───┐
     │Worker_2│          │tmp.2.2│          │tmp.2.7│
     └────────┘          └───────┘          └───────┘
```

Once tasks are finished, Worker_1 and Worker_2 will notify the Coordinator.
In case there are other tasks still pending in the Coordinator's queue, it will prompt the workers to request new tasks.
```text
     ┌──────┐            ┌───────────┐
     │Worker│            │Coordinator│
     └──┬───┘            └─────┬─────┘
        │    task completed    │      
        │ ─────────────────────>      
        │                      │      
        │ there remaining tasks│      
        │ <─────────────────────      
        │                      │      
        │     task request     │      
        │ ─────────────────────>      
     ┌──┴───┐            ┌─────┴─────┐
     │Worker│            │Coordinator│
     └──────┘            └───────────┘
```
If no more tasks are available, the Coordinator will also notify the workers to cease requesting tasks.
```text
     ┌──────┐                ┌───────────┐
     │Worker│                │Coordinator│
     └──┬───┘                └─────┬─────┘
        │      task completed      │      
        │ ─────────────────────────>      
        │                          │      
        │       no more task       │      
        │ <─────────────────────────      
        │                          │      
        │────┐                            
        │    │ cease requesting task      
        │<───┘                            
     ┌──┴───┐                ┌─────┴─────┐
     │Worker│                │Coordinator│
     └──────┘                └───────────┘
```