---
weight: 1
bookFlatSection: false
title: "TIPC"
bookToc: false
---

# TIPC
# 1. Introduction
Transparent Inter-Process Communication (TIPC) is an Unix Domain Sockets for transmitting data between nodes in a cluster. TIPC provides transparent communication, meaning that applications can communicate without needing to know the network addresses or locations of other processes.

Headers:
- /usr/include/linux/tipc.h

# 2. Addressing
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
                        __u32 domain;
                } name;
        } addr;
};
```
<--->
## family
```c++
#define AF_TIPC         30
#define PF_TIPC         AF_TIPC
#define SOL_TIPC        271
```

## addrtype
```c++
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
## tipc_socket_addr
```c++
struct tipc_socket_addr {
        __u32 ref;
        __u32 node;
};
```
A reference to a specific socket in the cluster.
<--->


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
{{< /columns >}}

## Sample: Bind a service address
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

## Sample: Bind a range of services
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

# Troubleshooting
**Unable to get TIPC nl family id (module loaded?)**
```sh
$ sudo modprobe tipc
$ lsmod | grep tipc
```
