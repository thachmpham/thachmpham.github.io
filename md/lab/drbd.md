---
title: 'Distributed Replicated Block Device (DRBD)'
subtitle: '(Lab Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- Create 2 QEMU VMs: VM1, VM2
- Connect VMs through a bridge: br0.
- Setup DRBD for the VMs.

:::
::: {.column}

```go
        VM1                 VM2
    192.0.0.10           192.0.0.20
       eth1                 eth1
        +                    +
        +--------------------+
                  +
                 br0
              192.0.0.254
               Container
```

:::
::::::::::::::


# Setup
## Setup Container
```sh
host$ git clone https://github.com/thachmpham/lab.git
host$ cd lab/drbd/setup-1
```

```sh
host$ docker compose build
host$ docker compose up --detach
host$ docker exec -it apk bash
```
