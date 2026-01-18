---
title: TIPC Protocol
subtitle: Transparent Inter-Process Communication
---

## 1. Setup Lab
### Enable TIPC
```sh

# host
$ modprobe tipc
$ lsmod | grep tipc
  
```

### Build Docker Image
```Dockerfile
  
# Dockerfile

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

### Run Docker Containers
```sh
  
$ docker run --privileged -dit --name node1 --hostname node1 tipc
$ docker run --privileged -dit --name node2 --hostname node2 tipc
  
$ docker exec -it node1 bash
$ docker exec -it node2 bash
  
```

### Attach Interface
```sh
  
# node1
$ tipc bearer enable media eth dev eth0

# node2
$ tipc bearer enable media eth dev eth0
  
```

### Hello World
```sh
  
# node1
$ /root/tipcutils/demos/hello_world/hello_server

# node2
$ /root/tipcutils/demos/hello_world/hello_client
  
```