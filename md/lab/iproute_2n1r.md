---
title: 'Iproute: 2 Nodes + 1 Router'
subtitle: '(Lab Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- Each node is a docker container:
    - H1: Node, host 1.
    - H2: Node, host 2.
    - RT: Router, forward messages between node.
- Allow transfer messages through the below paths:
    - H1 → RT → H2.
    - H2 → RT → H1.

:::
::: {.column}

```go
+--------------------+                     +----------------------+
| H1                 |                     |                   H2 |
|             eth0 + |                     | + eth0               |
|      10.0.0.2/24 | |                     | | 20.0.0.2/24        |
+------------------|-+                     +-|--------------------+
                   |                         |
+------------------|-------------------------|--------------------+
| RT               |                         |                    |
|             eth0 +                         + eth1               |
|      10.0.2.254/24                         20.0.0.254/24        |
+-----------------------------------------------------------------+
```

:::
::::::::::::::


# Setup
## Create Cluster
- Clone code.
```sh
host$ git clone https://github.com/thachmpham/lab.git
host$ cd lab/iproute2/2n1r
```


- Build nodes, networks.
```sh
host$ docker compose build
host$ docker compose up --detach
```

## Check Connection 
- Check h1.
```sh
host$ docker exec -it h1 bash
```

```sh
h1$ ip -br addr
eth0@if128       UP             10.0.0.2/24

h1$ traceroute -n 20.0.0.2
 1  10.0.0.254  0.705 ms  0.560 ms  0.529 ms
 2  20.0.0.2  0.506 ms  0.402 ms  0.367 ms

h1$ ping 20.0.0.2
64 bytes from 20.0.0.2: icmp_seq=1 ttl=63 time=0.133 ms
```


- Check h2.
```sh
host$ docker exec -it h2 bash
```

```sh
h2$ ip -br addr
eth0@if127       UP             20.0.0.2/24

h2$ traceroute -n 10.0.0.2
 1  20.0.0.254  1.501 ms  1.291 ms  1.257 ms
 2  10.0.0.2  1.233 ms  1.160 ms  1.126 ms

h2$ ping 10.0.0.2
64 bytes from 10.0.0.2: icmp_seq=1 ttl=63 time=0.314 ms
```

- Check rt.
```sh
$ docker exec -it rt bash
```

```sh
rt$ ip -br addr
eth0@if129       UP             10.0.0.254/24
eth1@if130       UP             20.0.0.254/24

rt$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```


# References
- [Kernel Selftest router.sh](https://github.com/torvalds/linux/blob/master/tools/testing/selftests/net/forwarding/router.sh)