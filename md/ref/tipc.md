---
title: TIPC Protocol
subtitle: Transparent Inter-Process Communication
---

# Setup Lab

:::::::::::::: {.columns}
::: {.column}

1. Dockerfile.
```Dockerfile
FROM ubuntu:24.04

RUN apt -y update
RUN apt install -y iproute2 tcpdump wget vim rsyslog
RUN apt install -y build-essential autoconf automake libtool pkg-config git
RUN apt install -y libnl-3-dev libnl-genl-3-dev libdaemon-dev libmnl-dev

RUN cd /root \
    && wget https://sourceforge.net/projects/tipc/files/tipcutils-3.0.6.tgz \
    && tar xf tipcutils-3.0.6.tgz \
    && mv tipcutils-3.0.6 tipcutils

WORKDIR /root/tipcutils

RUN ./bootstrap
RUN ./configure
RUN make
RUN make install

CMD ["tail", "-f", "/dev/null"]
```

```sh
$ docker build --progress=plain .
```

:::
::: {.column}

2. Host: Enable TIPC.
```sh
# host
$ modprobe tipc
$ lsmod | grep tipc
```

3. Run docker containers (test nodes).
```sh
$ docker run --privileged -dit --name node1 --hostname node1 tipc
$ docker run --privileged -dit --name node2 --hostname node2 tipc
  
$ docker exec -it node1 bash
$ docker exec -it node2 bash
```

4. Node 1: Setup bearer.
```sh
$ tipc bearer enable media eth dev eth0
```

5. Node 2: Setup bearer.
```sh
$ tipc bearer enable media eth dev eth0
```

:::
::::::::::::::

# Hello World

:::::::::::::: {.columns}
::: {.column}

Node 1.
```sh
$ /root/tipcutils/demos/hello_world/hello_server
```

:::
::: {.column}

Node 2.
```sh
$ /root/tipcutils/demos/hello_world/hello_client
```

:::
::::::::::::::

Decode strace output.

```sh
$ strace -yy -f -x -p `pidof hello_server`
```