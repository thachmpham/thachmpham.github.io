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
Decode strace to find tipc address type and instance on recvfrom and sendto syscalls.

1. Collect strace.
```sh
$ /root/tipcutils/demos/hello_world/hello_server
$ /root/tipcutils/demos/hello_world/hello_client
```

```sh
$ strace -yy -x --read=3 --write=3 -f -p `pidof hello_server`

strace: Process 22515 attached
recvfrom(3<socket:[91081]>, "\x48\x65\x6c\x6c\x6f\x20\x57\x6f\x72\x6c\x64\x21\x21\x21\x00", 40, 0, {sa_family=AF_TIPC, sa_data="\x03\x00\xbd\xc9\xc3\x1b\x3b\x27\xe1\x59\x00\x00\x00\x00"}, [16]) = 15
 | 00000  48 65 6c 6c 6f 20 57 6f  72 6c 64 21 21 21 00     Hello World!!!.  |
write(1</dev/pts/2<char 136:2>>, "Server: Message received: Hello "..., 42) = 42
sendto(3<socket:[91081]>, "\x55\x68\x20\x3f\x00", 5, 0, {sa_family=AF_TIPC, sa_data="\x03\x00\xbd\xc9\xc3\x1b\x3b\x27\xe1\x59\x00\x00\x00\x00"}, 16) = 5
 | 00000  55 68 20 3f 00                                    Uh ?.            |
write(1</dev/pts/2<char 136:2>>, "Server: Sent response : Uh ? \n", 30) = 30
write(1</dev/pts/2<char 136:2>>, "****** TIPC hello world server f"..., 47) = 47
```

:::::::::::::: {.columns}
::: {.column}

2. Strace raw data.
```c
{sa_family=AF_TIPC, sa_data="\x03\x00\xbd\xc9\xc3\x1b\x3b\x27\xe1\x59\x00\x00\x00\x00"}
```

3. Check tipc headers to determine the format.
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

4. Decode.
```sh
$ python3 decode.py '\x03\x00\xbd\xc9\xc3\x1b\x3b\x27\xe1\x59\x00\x00\x00\x00'
sa_data(addrtype=3, scope=0, type=465815997, instance=1507927867)
```

:::
::::::::::::::