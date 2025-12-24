---
title:  'PlantUML'
---

:::::::::::::: {.columns}
::: {.column width=50%}

## Docker

Pull docker image.
```sh
$ docker pull plantuml/plantuml-server:jetty
```

Run container.
```sh
$ docker run -d -p 8080:8080 plantuml/plantuml-server:jetty  
```

:::
::: {.column width=50%}

## Jar

Download jar.
```sh
$ wget https://github.com/plantuml/plantuml/releases/download/v1.2025.10/plantuml-1.2025.10.jar
```

Generate image.
```sh
$ java -jar plantuml-1.2025.10.jar hello.puml
```

:::
::::::::::::::


## Useful Options

:::::::::::::: {.columns}
::: {.column}

[Theme](https://plantuml.com/theme), [Handwritten](https://plantuml.com/handwritten).
```yml
!theme plain
!option handwritten true
```

Good themes: plain, sketchy, sketchy-outline.

:::
::: {.column}

[Sequence Diagram](https://plantuml.com/sequence-diagram)
```yml
!pragma teoz true
hide footbox
```

:::
::::::::::::::