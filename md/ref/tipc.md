---
title: TIPC Protocol
subtitle: Transparent Inter-Process Communication
---

# 1. Setup Lab

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


# 2. Hello World

:::::::::::::: {.columns}
::: {.column width=50%}

- Node 1: Start server.
```sh
$ /root/tipcutils/demos/hello_world/hello_server
```

:::
::: {.column width=50%}

- Node 2: Start client.
```sh
$ /root/tipcutils/demos/hello_world/hello_client
```

:::
::::::::::::::


# 3. Decode Strace
Decode sa_data in strace to find tipc addresses.

1. Collect strace.
```sh
$ strace -e trace=network -yy -x -f ./hello_server

****** TIPC hello world server started ******
socket(AF_TIPC, SOCK_RDM, 0)            = 3<socket:[106808]>
bind(3<socket:[106808]>, {sa_family=AF_TIPC, sa_data="\x02\x02\xc8\x49\x00\x00\x11\x00\x00\x00\x00\x00\x00\x00"}, 16) = 0
recvfrom(3<socket:[106808]>, "\x48\x65\x6c\x6c\x6f\x20\x57\x6f\x72\x6c\x64\x21\x21\x21\x00", 40, 0, {sa_family=AF_TIPC, sa_data="\x03\x00\x7a\xe4\x1a\xbb\x00\x00\x00\x00\x00\x00\x00\x00"}, [16]) = 15
Server: Message received: Hello World!!! 
sendto(3<socket:[106808]>, "\x55\x68\x20\x3f\x00", 5, 0, {sa_family=AF_TIPC, sa_data="\x03\x00\x7a\xe4\x1a\xbb\x00\x00\x00\x00\x00\x00\x00\x00"}, 16) = 5
Server: Sent response : Uh ? 
****** TIPC hello world server finished ******
+++ exited with 0 +++
```

```sh
$ strace -e trace=network -yy -x -f ./hello_client

****** TIPC hello world client started ******
socket(AF_TIPC, SOCK_SEQPACKET, 0)      = 3<socket:[108592]>
connect(3<socket:[108592]>, {sa_family=AF_TIPC, sa_data="\x02\x00\x01\x00\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00"}, 16) = 0
sendto(3<socket:[108592]>, "\xc8\x49\x00\x00\x11\x00\x00\x00\x11\x00\x00\x00\x10\x27\x00\x00\x02\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00", 28, 0, NULL, 0) = 28
recvfrom(3<socket:[108592]>, "\x01\x00\x00\x00\x11\x00\x00\x00\x11\x00\x00\x00\xb7\x38\xcd\x18\x00\x00\x00\x00\xc8\x49\x00\x00\x11\x00\x00\x00\x11\x00\x00\x00"..., 48, 0, NULL, NULL) = 48
socket(AF_TIPC, SOCK_RDM, 0)            = 3<socket:[108593]>
sendto(3<socket:[108593]>, "\x48\x65\x6c\x6c\x6f\x20\x57\x6f\x72\x6c\x64\x21\x21\x21\x00", 15, 0, {sa_family=AF_TIPC, sa_data="\x02\x00\xc8\x49\x00\x00\x11\x00\x00\x00\x00\x00\x00\x00"}, 16) = 15
Client: sent message: Hello World!!! 
recvfrom(3<socket:[108593]>, "\x55\x68\x20\x3f\x00", 40, 0, NULL, NULL) = 5
Client: received response: Uh ? 
****** TIPC hello client finished ******
+++ exited with 0 +++
```

:::::::::::::: {.columns}
::: {.column}

2. Check tipc headers to determine the format.
```c
// header: /usr/include/linux/tipc.h
struct sockaddr_tipc {
	unsigned short family;      //0     2   already decode by strace
	unsigned char  addrtype;    //2     1   (B)
	signed   char  scope;       //3     1   (b)
	union {                     //4     12
		struct tipc_socket_addr id; //      (II)
		struct tipc_service_range nameseq;
		struct {
			struct tipc_service_addr name;
			__u32 domain;
		} name;
	} addr;             
};

struct tipc_service_addr {
	__u32 type;                 //0     4   (I)        
	__u32 instance;             //4     4   (I)
};
```

:::
::: {.column}

3. decode.py
```python
# Usage:
#   python3 decode.py '\x03\x00\xbd\xc9\xc3\x1b\x3b\x27\xe1\x59\x00\x00\x00\x00'

import struct, sys
from collections import namedtuple

raw = bytes(sys.argv[1], 'utf-8').decode('unicode_escape').encode('latin1')

fmt = '=BbII4x'
data = struct.unpack(fmt, raw)

names = namedtuple('sa_data', 'addrtype scope type instance')
record = names._make(data)

print(record)
```

:::
::::::::::::::


:::::::::::::: {.columns}
::: {.column}

4. Decode server strace.
```sh
# bind
$ python3 decode.py '\x02\x02\xc8\x49\x00\x00\x11\x00\x00\x00\x00\x00\x00\x00'
sa_data(addrtype=3, scope=0, type=465815997, instance=1507927867)

# recvfrom
$ python3 decode.py '\x03\x00\x7a\xe4\x1a\xbb\x00\x00\x00\x00\x00\x00\x00\x00'
sa_data(addrtype=3, scope=0, type=3139101818, instance=0)

# sendto
$ python3 decode.py '\x03\x00\x7a\xe4\x1a\xbb\x00\x00\x00\x00\x00\x00\x00\x00'
sa_data(addrtype=3, scope=0, type=3139101818, instance=0)
```

:::
::: {.column}

5. Decode client strace.
```sh
# connect
$ python3 decode.py '\x02\x00\x01\x00\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00'
sa_data(addrtype=2, scope=0, type=1, instance=1)

# sendto
# recvfrom

# sendto
$ python3 decode.py '\x02\x00\xc8\x49\x00\x00\x11\x00\x00\x00\x00\x00\x00\x00'
sa_data(addrtype=2, scope=0, type=18888, instance=17)

# recvfrom
```

:::
::::::::::::::