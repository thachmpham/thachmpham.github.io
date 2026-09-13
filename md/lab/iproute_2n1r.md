---
title: 'Iproute: 2 Nodes + 1 Router'
---


# Overview

:::::::::::::: {.columns}
::: {.column}

- Each node is a docker container:
    - H1: Host 1.
    - H2: Host 2.
    - RT: Router, forward messages between hosts.
- Allow transfer messages through the below paths:
    - H1 → RT → H2.
    - H2 → RT → H1.

:::
::: {.column}

```go
+--------------------+                     +----------------------+
| H1                 |                     |                   H2 |
|            $eth0 + |                     | + $eth0              |
|      10.0.0.2/24 | |                     | | 20.0.0.2/24        |
|                  | |                     | |                    |
+------------------|-+                     +-|--------------------+
                   |                         |
+------------------|-------------------------|--------------------+
| RT (router)      |                         |                    |
|            $eth0 +                         + $eth1              |
|      10.0.2.100/24                         20.0.0.100/24        |
|                                                                 |
+-----------------------------------------------------------------+
```

:::
::::::::::::::


# Setup

- Clone code.
```sh
host$ git clone https://github.com/thachmpham/lab.git
host$ cd lab/iproute2/2n1r
```

- Build nodes.
```sh
host$ docker compose build
host$ docker compose up --detach
```

- Check h1.
```sh
host$ docker exec -it h1 bash
```

```sh
h1$ ip route show
default via 10.0.0.1 dev eth0
10.0.0.0/24 dev eth0 proto kernel scope link src 10.0.0.2
20.0.0.0/24 via 10.0.0.100 dev eth0

h1$ ip route get 20.0.0.2
20.0.0.2 via 10.0.0.100 dev eth0 src 10.0.0.2 uid 0

h1$ ping -c3 20.0.0.2
64 bytes from 20.0.0.2: icmp_seq=1 ttl=63 time=0.178 ms

h1$ traceroute -n 20.0.0.2
 1  10.0.0.100  1.301 ms  1.119 ms  1.071 ms
 2  20.0.0.2  1.031 ms  0.931 ms  0.877 ms
```

- Check h2.
```sh
host$ docker exec -it h2 bash
```

```sh
h2$ ip route show
default via 20.0.0.1 dev eth0
10.0.0.0/24 via 20.0.0.100 dev eth0
20.0.0.0/24 dev eth0 proto kernel scope link src 20.0.0.2

h2$ ip route get 10.0.0.2
10.0.0.2 via 20.0.0.100 dev eth0 src 20.0.0.2 uid 0

h2$ ping -c3 10.0.0.2
64 bytes from 10.0.0.2: icmp_seq=1 ttl=63 time=0.258 ms

h2$ traceroute -n 10.0.0.2
 1  20.0.0.100  1.403 ms  1.228 ms  1.182 ms
 2  10.0.0.2  1.141 ms  0.977 ms  0.920 ms
```


# References
- [Kernel Selftest router.sh](https://github.com/torvalds/linux/blob/master/tools/testing/selftests/net/forwarding/router.sh)