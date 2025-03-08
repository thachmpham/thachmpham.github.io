---
title:  'PlantUML'
---

# 1. Install
- Pull docker image.
```sh
  
$ docker pull plantuml/plantuml-server:jetty
  
```

- Run container.
```sh
  
$ docker run -d -p 8080:8080 plantuml/plantuml-server:jetty
  
```


# 2. Diagrams
## 2.1. Sequence Diagram
```python
  
@startuml

hide footbox
title Sample Sequence

participant Alice
participant Bob

Alice -> Bob:   Request
Bob --> Alice:  Reply

@enduml
  
```


## 2.2. Class Diagram
```python
  
@startuml

class Dummy {
    filed1
    method1()
}

@enduml
  
```


## 2.3. Activity Diagram
```python
  
@startuml

start

if (Graphviz installed?) then (yes)
  :process all\ndiagrams;
else (no)
  :process only
  __sequence__ and __activity__ diagrams;
endif

stop

@enduml
  
```


## 2.3. Gantt Diagram
```python
  
@startgantt
Project starts 2020-07-01
[Prototype design] starts 2020-07-01 and ends 2020-07-15
[Test prototype] starts 2020-07-16 and requires 10 days
@endgantt
  
```


## 2.3. Mindmap
```python
  
@startmindmap
* Debian
** Ubuntu
*** Linux Mint
*** Kubuntu
*** Lubuntu
*** KDE Neon
** LMDE
** SolydXK
** SteamOS
** Raspbian with a very long name
*** <s>Raspmbc</s> => OSMC
*** <s>Raspyfi</s> => Volumio
@endmindmap
  
```

# References
- [plantuml.com](https://plantuml.com)