---
title:  'Graphviz'
---


# 1. Install
- Ubuntu
```sh
  
$ apt install graphviz
  
```

- Fedora
```sh
  
$ dnf install graphviz
  
```


# 2. Command
```sh
  
$ dot -Tpng hello.dot -o hello.png
  
```


# 3. Graphs
## 3.1. Directed Graph
```python
  
digraph G {
    A -> B
    B -> C
}
  
```


## 3.2. Undirected Graph
```python
  
graph G {
    A -- B
    B -- C
}
  
```


## 3.3. Subgraph
```python
  
digraph G {
	subgraph cluster_0 {
		style=filled;
		color=lightgrey;
        label = "cluster 1";
		
		a0 -> a1 -> a2 -> a3;
		
	}

	subgraph cluster_1 {
		label = "cluster 2";
		
		b0 -> b1 -> b2 -> b3;
	}

	a1 -> b3;
	b2 -> a3;
	a3 -> a0;

}
  
```


# References
- [graphviz.org](https://graphviz.org)
- [magjac.com/graphviz-visual-editor](https://magjac.com/graphviz-visual-editor)