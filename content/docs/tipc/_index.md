---
weight: 1
title: "TIPC"
bookToc: false
bookFlatSection: true
---

# TIPC
# 1. Introduction
TIPC (Transparent Inter-Process Communication) is a communication protocol used for inter-process communication (IPC) in distributed systems. It enables processes running on different nodes within a network to communicate with each other transparently, abstracting away the underlying network details.

Headers:
- /usr/include/linux/tipc.h

Websites:
- tipc.sourceforge.net


# 2. Setup A Cluster
A TIPC cluster consists of nodes interconnected with links. A node can be either a physical processor, a virtual machine or a network namespace.
```
      Node 1                                             Node 2
-----------------                                 -----------------
|      TIPC       |                               |      TIPC       |
|   Application   |                               |   Application   |
|-----------------|                               |-----------------|
|                 |                               |                 |
|      TIPC       |TIPC address       TIPC address|      TIPC       |
|                 |                               |                 |
|-----------------|                               |-----------------|
|  Bearer Service |Bearer address   Bearer address|  Bearer Service |
|                 |                               |                 |
-----------------                                 -----------------
        |                                                  |
        +---------------- Bearer Transport ----------------+
```

## 2.1. Setup Node Identities
{{< columns >}}
### Node 1
```sh
# tipc node set identity <node_id>
$ tipc node set identity node_1

$ tipc node get identity
Node Identity                    Hash
node_1                           00000000
```

<--->

### Node 2
```sh
# tipc node set identity <node_id>
$ tipc node set identity node_2

$ tipc node get identity
Node Identity                    Hash
node_2                           00000000
```

{{< /columns >}}


## 2.2. Setup Bearers
A bearer is an abstraction of a network interface that transmits message processes or nodes in a TIPC network.
{{< columns >}}

### Node 1
```sh
# Find network interfaces
$ ifconfig
ens160: inet 172.16.111.128  netmask 255.255.255.0  broadcast 172.16.111.255

# Enable bear on a network interface
$ tipc bear enable media eth device ens160

$ tipc bear list
eth:ens160
```

<--->

### Node 2
```sh
# Find network interfaces
$ ifconfig
ens160: inet 172.16.111.129  netmask 255.255.255.0  broadcast 172.16.111.255

# Enable bear on a network interface
$ tipc bear enable media eth device ens160

$ tipc bear list
eth:ens160
```

{{< /columns >}}

## 2.3. Show Nametable
Nametable contains information about the network topology, including the identities of nodes and services within the TIPC cluster.

```sh
$ tipc nametable show
Type       Lower      Upper      Scope    Port       Node
0          828204133  828204133  cluster  0          node_2
0          828269669  828269669  cluster  0          node_1
1          1          1          node     3236269336 node_2
2          828269669  828269669  node     0          node_2
```

- The two entries with service type 0 show that we have two nodes in the cluster.
- The entry with service type 1 represents the built-in topology (service tracking) service.
- The entry with service type 2 show the link.


# 3. Hello World
## 3.1. Install Packages
```sh
$ sudo apt-get install build-essential
$ sudo apt-get install autoconf
```

## 3.2. Build Tipcutils
```sh
$ git clone https://github.com/TIPC/tipcutils.git
$ cd tipcutils
$ ./bootstrap
$ ./configure
$ make
$ sudo make install
```

## 3.3. Run Hello World

{{< columns >}}

### Node 1
Start a tipc server with type:18888, instance:17.
```sh
$ cd tipcutils/demos/server_tipc
$ ./server_tipc
****** TIPC server hello world program started ******

Server: Message received: Hello World !

****** TIPC server hello program finished ******
```

<--->

### Node 2
Start a client that connect to the server.
```sh
$ cd tipcutils/demos/client_tipc
$ ./client_tipc
****** TIPC client hello world program started ******

Client: received response: Uh ?

****** TIPC client hello program finished ******
```

{{< /columns >}}

## 3.4. Show Nametable
Start server in a terminal.
```sh
$ cd tipcutils/demos/server_tipc
$ ./server_tipc
```

Show nametable in another terminal.
```sh
$ tipc nametable show
Type       Lower      Upper      Scope    Port       Node
18888      17         17         cluster  3000685032 node_1
```
- The entry with service type 18888 show the server.

# X. Addressing
{{< columns >}}
## sockaddr_tipc
```c++
struct sockaddr_tipc {
        unsigned short family;
        unsigned char  addrtype;
        signed   char  scope;
        union {
                struct tipc_socket_addr id;
                struct tipc_service_range nameseq;
                struct {
                        struct tipc_service_addr name;
                        __u32 domain; // lookup scope: valid node hash number or zero.

                } name;
        } addr;
};
```
<--->
## family
```c++
#define AF_TIPC         30
```

## addrtype
```c++
#define TIPC_ADDR_MCAST         1
#define TIPC_SERVICE_RANGE      1
#define TIPC_SERVICE_ADDR       2
#define TIPC_SOCKET_ADDR        3
```

## scope
```c++
enum tipc_scope {
        TIPC_CLUSTER_SCOPE = 2,
        TIPC_NODE_SCOPE    = 3
};
```
{{< /columns >}}


{{< columns >}}
## tipc_service_addr
```c++
struct tipc_service_addr {
        __u32 type;
        __u32 instance;
};
```
ID of a service within the TIPC network. The service types 0 through 63 are reserved for system internal use, and are not available for user space applications.

<--->

## tipc_service_range
```c++
struct tipc_service_range {
        __u32 type;
        __u32 lower;
        __u32 upper;
};
```
A range of service addresses of the same type and with instances between a lower and an upper range limit.

<--->

## tipc_socket_addr
```c++
struct tipc_socket_addr {
        __u32 ref;
        __u32 node;
};
```
A reference to a specific socket in the cluster.

{{< /columns >}}


## Samples

{{< tabs "uniqueid" >}}
{{< tab "tipc_service_addr" >}}

Bind a TIPC service with type is **100** and instance ID is **1**.

```c++
#include <sys/types.h>
#include <sys/socket.h>
#include <linux/tipc.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>

int main(int argc, char *argv[])
{
	int service_type = 100;
	int instance = 1;

	struct sockaddr_tipc server = {
		.family = AF_TIPC,
		.addrtype = TIPC_SERVICE_ADDR,
		.scope = TIPC_CLUSTER_SCOPE,
		.addr.name.name.type = service_type,
		.addr.name.name.instance = instance
	};

	int sd = socket(AF_TIPC, SOCK_RDM, 0);

	if (0 != bind(sd, (void*)&server, sizeof(server))) {
		printf("Bind failed, error=%s\n", strerror(errno));
		exit(1);
	}

	// Keep process running
	printf("Bind successfully, press Enter to exit...\n");
	while ((getchar()) != '\n') {}
}
```

```sh
$ gcc demo.c -o demo

$ ./demo
Bind service to type=100, instance=1 successfully
Press Enter to exit
```

```sh
$ tipc nametable show
Type       Lower      Upper      Scope    Port       Node
100        1          1          cluster  1603289447 1001002
```
{{< /tab >}}

{{< tab "tipc_service_range" >}}

Bind a TIPC service with type is **100** and range is **0-10**.
```c++
#include <sys/types.h>
#include <sys/socket.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <linux/tipc.h>

int main(int argc, char* argv[])
{
    int service_type = 100;
    int lower = 0;
    int upper = 10;

	struct sockaddr_tipc server_addr;
	server_addr.family = AF_TIPC;
	server_addr.addrtype = TIPC_SERVICE_RANGE;
	server_addr.scope = TIPC_CLUSTER_SCOPE;
	server_addr.addr.nameseq.type = service_type;
	server_addr.addr.nameseq.lower = lower;
	server_addr.addr.nameseq.upper = upper;

	int sd = socket(AF_TIPC, SOCK_RDM, 0);

	if (bind(sd, (struct sockaddr *)&server_addr, sizeof(server_addr))) {
		printf("Bind failed, error=%s\n", strerror(errno));
		exit (1);
	}

	// Keep process running
	printf("Bind successfully, press Enter to exit\n");
	while ((getchar()) != '\n') {}
}
```

```sh
$ tipc nametable show
Type       Lower      Upper      Scope    Port       Node
100        0          10         cluster  2776395043
```

{{< /tab >}}
{{< /tabs >}}

# X. Troubleshooting
**Unable to get TIPC nl family id (module loaded?)**
```sh
$ sudo modprobe tipc
$ lsmod | grep tipc
```

**socket(): Address family not supported by protocol**
```sh
$ sudo modprobe tipc
```