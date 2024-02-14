---
weight: 1
bookFlatSection: false
title: "TIPC"
bookToc: false
---

# TIPC
## 1. Introduction
Transparent Inter-Process Communication (TIPC) is an Unix Domain Sockets for transmitting data between nodes in a cluster.

Headers:
- /usr/include/linux/tipc.h

## 2. Addressing
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
<--->
## tipc_service_addr
```c++
struct tipc_service_addr {
        __u32 type;
        __u32 instance;
};
```
The service types 0 through 63 are reserved for system internal use, and are not available for user space applications.
<--->
## tipc_service_range
```c++
struct tipc_service_range {
        __u32 type;
        __u32 lower;
        __u32 upper;
};
```
{{< /columns >}}

## Sample: bind a service address
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
		printf("Server: failed to bind port name, error=%s\n", strerror(errno));
		exit(1);
	}

	printf("Bind service to type=%d, instance=%d successfully\n", service_type, instance);

	printf("Press Enter to exit\n");
	while ((getchar()) != '\n') {}
}
```

{{< columns >}}
```sh
$ gcc demo.c -o demo

$ ./demo
Bind service to type=100, instance=1 successfully
Press Enter to exit
```
<--->
```sh
$ tipc nametable show
Type       Lower      Upper      Scope    Port       Node
0          16781314   16781314   cluster  0          1001002
1          1          1          node     2873835490 1001002
100        1          1          cluster  1603289447 1001002
```
{{< /columns >}}
