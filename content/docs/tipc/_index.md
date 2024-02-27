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

Implementation:
- https://github.com/torvalds/linux/tree/master/net/tipc

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

In this lab, node 1 and node 2 are virtual machines connected with each other.


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

{{< columns >}}
- The two entries with service type 0 show that we have two nodes in the cluster.
- The entry with service type 1 represents the built-in topology (service tracking) service.
- The entry with service type 2 show the link.

<--->
```c++
// /usr/include/linux/tipc.h
#define TIPC_NODE_STATE	0	/* node state service type */
#define TIPC_TOP_SRV		1	/* topology server service type */
#define TIPC_LINK_STATE	2	/* link state service type */
```

{{< /columns >}}

# 3. Hello World
## 3.1. Install Packages
```sh
$ sudo apt-get install build-essential 
$ sudo apt-get install autoconf libtool pkg-config libdaemon-dev libmnl-dev
```

## 3.2. Build Tipcutils
```sh
$ wget https://sourceforge.net/projects/tipc/files/tipcutils_3.0.6.tgz
$ tar xvf tipcutils_3.0.6.tgz
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
$ cd tipcutils/demos/hello_world
$ ./hello_server
****** TIPC server hello world program started ******

Server: Message received: Hello World !

****** TIPC server hello program finished ******
```

<--->

### Node 2
Start a client that connect to the server.
```sh
$ cd tipcutils/demos/hello_world
$ ./hello_client
****** TIPC client hello world program started ******

Client: received response: Uh ?

****** TIPC client hello program finished ******
```

{{< /columns >}}

## 3.4. Show Nametable
Start server in a terminal.
```sh
$ cd tipcutils/demos/server_tipc
$ ./hello_server
```

Show nametable in another terminal.
```sh
$ tipc nametable show
Type       Lower      Upper      Scope    Port       Node
18888      17         17         cluster  3000685032 node_1
```
- The entry with service type 18888 show the server.

## 3.5. Code
{{< tabs "helloworld" >}}

{{< tab "hello_server.c" >}}
```c++
// start server
struct sockaddr_tipc server = {
		.family = AF_TIPC,
		.addrtype = TIPC_SERVICE_ADDR,
		.scope = TIPC_CLUSTER_SCOPE,
		.addr.name.name.type = 18888,
		.addr.name.name.instance = 17
};
sd = socket(AF_TIPC, SOCK_RDM, 0);
bind(sd, &server, sizeof(server));

// communicate with client
struct sockaddr_tipc client;
recvfrom(sd, inbuf, sizeof(inbuf), 0, &client, &alen));
sendto(sd, outbuf, strlen(outbuf) + 1, 0, &client, sizeof(client));
```
{{< /tab >}}


{{< tab "hello_client.c" >}}
```c++
// connect to topology server
struct sockaddr_tipc topsrv = {
		.family = AF_TIPC,
		.addrtype = TIPC_SERVICE_ADDR,
		.addr.name.name.type = TIPC_TOP_SRV,
		.addr.name.name.instance = TIPC_TOP_SRV,
		.addr.name.domain = 0
};
sd = socket(AF_TIPC, SOCK_SEQPACKET, 0);
connect(sd, &topsrv, sizeof(topsrv)))

// subscribe with topology server to receive events from hello_server
struct tipc_subscr subscr = {
		.seq.type = 18888,
		.seq.lower = 17,
		.seq.upper = 17,
		.timeout = 10000, // 10 seconds
		.filter = TIPC_SUB_SERVICE
};
send(sd, &subscr, sizeof(subscr), 0)

// wait for the subscription to fire or wait for server become alive
recv(sd, &event, sizeof(event), 0)

// communicate with hello_server
struct sockaddr_tipc server = {
		.family = AF_TIPC,
		.addrtype = TIPC_SERVICE_ADDR,
		.addr.name.name.type = 18888,
		.addr.name.name.instance = 17,
		.addr.name.domain = 0
	};
socket(AF_TIPC, SOCK_RDM, 0);
sendto(sd, buf, strlen(buf) + 1, 0, &server, sizeof(server)))
recv(sd, buf, sizeof(buf), 0))
```
{{< /tab >}}

{{< /tabs >}}

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

{{< tabs "address_samples" >}}

{{< tab "tipc_service_addr" >}}

Bind a TIPC service with type is **100** and instance ID is **1**. Full code at [HERE](https://github.com/thachmpham/samples/blob/main/tipc/tipc_service_addr.c).

```c++
struct sockaddr_tipc server = {
	.family = AF_TIPC,
	.addrtype = TIPC_SERVICE_ADDR,
	.scope = TIPC_CLUSTER_SCOPE,
	.addr.name.name.type = 100,
	.addr.name.name.instance = 1
};

sd = socket(AF_TIPC, SOCK_RDM, 0);

bind(sd, &server, sizeof(server));
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

Bind a TIPC service with type is **100** and range is **0-10**. Full code at [HERE](https://github.com/thachmpham/samples/blob/main/tipc/tipc_service_range.c).
```c++
struct sockaddr_tipc server_addr;
server_addr.family = AF_TIPC;
server_addr.addrtype = TIPC_SERVICE_RANGE;
server_addr.scope = TIPC_CLUSTER_SCOPE;
server_addr.addr.nameseq.type = 100;
server_addr.addr.nameseq.lower = 0;
server_addr.addr.nameseq.upper = 10;

sd = socket(AF_TIPC, SOCK_RDM, 0);

bind(sd, (struct sockaddr *)&server_addr, sizeof(server_addr));
```

```sh
$ tipc nametable show
Type       Lower      Upper      Scope    Port       Node
100        0          10         cluster  2776395043
```

{{< /tab >}}

{{< tab "tipc_socket_addr" >}}

Binding a tipc_socket_addr is not allowed. More details at [HERE](https://github.com/torvalds/linux/blob/master/net/tipc/socket.c).
```c++
static int tipc_bind(_, struct sockaddr *skaddr, _)
{
	struct tipc_uaddr *ua = (struct tipc_uaddr *)skaddr;
	u32 atype = ua->addrtype;

	if (atype == TIPC_SOCKET_ADDR)
		return -EAFNOSUPPORT; // Address family not supported by protocol
}
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

**Reload tipc: all configures will be reset**
```sh
$ sudo modprobe -r tipc
$ sudo modprobe tipc
```
