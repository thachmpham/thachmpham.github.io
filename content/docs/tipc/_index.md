---
weight: 1
title: "TIPC"
bookToc: false
bookFlatSection: true
---

# TIPC
# 1. Introduction
Transparent Inter-Process Communication (TIPC) is a communication protocol used for inter-process communication (IPC) in distributed systems. It enables processes running on different nodes within a network to communicate with each other transparently, abstracting away the underlying network details.

Websites:
- [tipc.sourceforge.net](https://tipc.io)

Papers:
- [TIPC Communication Group Netdev 0x12 Paper](https://sourceforge.net/projects/tipc/files/TIPC%20Communication%20Groups%20Netdev%200x12%20Paper.pdf/download)

Implementation:
- [github.com/torvalds/linux/tree/master/net/tipc](https://github.com/torvalds/linux/tree/master/net/tipc)

Headers:
- /usr/include/linux/tipc.h

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

# 4. Addressing
## 4.1. API
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
TIPC_ZONE_SCOPE    = 1
TIPC_CLUSTER_SCOPE = 2
TIPC_NODE_SCOPE    = 3
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


## 4.2. Samples

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
// net/tipc/socket.c
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


# 5. Service & Topology Tracking
TIPC provides a service tracking function that makes it possible for an application to follow the availability of service addresses and service ranges in the cluster.

## 5.1. API
An application accesses the topology service by opening a SOCK_SEQPACKET type connection to the TIPC internal topology service, using the service address {TIPC_TOP_SRV, TIPC_TOP_SRV}. In return, the topology service sends service event messages back to the application whenever matching addresses are bound or unbound by sockets within the cluster.

{{< columns >}}
## tipc_subscr
```c++
struct tipc_subscr {
        struct tipc_service_range seq;  /* range of interest */
        __u32 timeout;                  /* subscription duration (in ms) */
        __u32 filter;                   /* bitmask of filter options */
        char usr_handle[8];             /* available for subscriber use */
};
```

<--->

## filter
```c++
#define TIPC_SUB_PORTS          0x01    /* filter: evt at each match */
#define TIPC_SUB_SERVICE        0x02    /* filter: evt at first up/last down */
#define TIPC_SUB_CANCEL         0x04    /* filter: cancel a subscription */
```

{{< /columns >}}


{{< columns >}}
## tipc_subscr
```c++
struct tipc_event {
        __u32 event;                    /* event type */
        __u32 found_lower;              /* matching range */
        __u32 found_upper;              /*    "      "    */
        struct tipc_socket_addr port;   /* associated socket */
        struct tipc_subscr s;           /* associated subscription */
};
```

<--->

## event
```c++
#define TIPC_PUBLISHED          1       /* publication event */
#define TIPC_WITHDRAWN          2       /* withdrawal event */
#define TIPC_SUBSCR_TIMEOUT     3       /* subscription timeout event */
```

{{< /columns >}}

## 5.2. Sample
### Build topology_subscr_demo
```sh
$ cd tipcutils/demos/topology_subscr_demo
$ make
```


### Run Client
```sh
$ ./top_client
Client: connected to topology server
Client: issued subscription to {18888,0,100}, timeout 500000, user ref: 2
Client: issued subscription to {0,0,4294967295}, timeout 4294967295, user ref: 3
Client: subscriptions remain active until client is killed

Client: received event for published {18888,6,53} port id <315e6465:4178109087>
associated with network node 315e6465
generated by subscription to {18888,0,100}, timeout 500000, user ref: 2

Client: received event for published {18888,54,55} port id <315e6465:4178109087>
associated with network node 315e6465
generated by subscription to {18888,0,100}, timeout 500000, user ref: 2

Client: received event for withdrawn {18888,54,55} port id <315e6465:4178109087>
associated with network node 315e6465
generated by subscription to {18888,0,100}, timeout 500000, user ref: 2

Client: received event for withdrawn {18888,6,53} port id <315e6465:4178109087>
associated with network node 315e6465
generated by subscription to {18888,0,100}, timeout 500000, user ref: 2
```

### Run Server
```sh
$ ./top_server
Server: bound port A to {18888,6,53} scope 1
Server: bound port A to name sequence {18888,6,53} scope 1
Server: bound port A to name sequence {18888,54,55} scope 1
Server: bound port B to name sequence {18888,54,55} scope 1

Server: port names remain published until server is killed
^C
```


## 5.3. Code
```c++
// connect to topology service
struct sockaddr_tipc topsrv;
topsrv.family = AF_TIPC;
topsrv.addrtype = TIPC_SERVICE_ADDR;
topsrv.addr.name.name.type = TIPC_TOP_SRV;
topsrv.addr.name.name.instance = TIPC_TOP_SRV;
sd = socket (AF_TIPC, SOCK_SEQPACKET, 0);
connect(sd, (struct sockaddr *)&topsrv, sizeof(topsrv))

// subscribe events from service type 18888
struct tipc_subscr subscr;
subscr.seq.type  = htonl(18888);
subscr.seq.lower = htonl(0);
subscr.seq.upper = htonl(100);
subscr.timeout   = 500000;  // miliseconds
subscr.filter    = htonl(TIPC_SUB_SERVICE);
send(sd, &subscr, sizeof(subscr), 0);

// wait for events
while (recv(sd, &event, sizeof(event), 0) == sizeof(event)) {
		print_evt("Client: received event for", &event);
}
```


# 6. Messaging
TIPC message transmission can be performed in different ways:
- Datagram: SOCK_DGRAM, SOCK_RDM.
- Connection oriented: SOCK_STREAM, SOCK_SEQPACKET.
- Group.

```c++
/* Types of sockets.  */
enum __socket_type
{
  SOCK_STREAM = 1,              /* Sequenced, reliable, connection-based byte streams.  */
  SOCK_DGRAM = 2,               /* Connectionless, unreliable datagrams of fixed maximum length.  */
  SOCK_RAW = 3,                 /* Raw protocol interface.  */
  SOCK_RDM = 4,                 /* Reliably-delivered messages.  */
  SOCK_SEQPACKET = 5,           /* Sequenced, reliable, connection-based, datagrams of fixed maximum length.  */
  SOCK_DCCP = 6,                /* Datagram Congestion Control Protocol.  */
};
```

## 6.1. Datagram Messaging
{{< columns >}}
### Client
```c++
sock_type = <SOCK_DGRAM or SOCK_RDM>
sd = socket(AF_TIPC, sock_type, 0);



sendto(sd, buf, strlen(buf), 0, &server, sizeof(server));
recv(sd, buf, sizeof(buf), 0)
```

```sh
$ cd tipcutils/demos/tipc-pipe
$ ./tipc-pipe --sock_type SOCK_DGRAM
```

<--->
### Server
```c++
sock_type = <SOCK_DGRAM or SOCK_RDM>
sd = socket(AF_TIPC, socket_type, 0);

bind(sd, &server, sizeof(server));

recvfrom(sd, inbuf, sizeof(inbuf), 0, &client, &alen));
sendto(sd, outbuf, strlen(outbuf), 0, &client, sizeof(client))
```

```sh
$ cd tipcutils/demos/tipc-pipe
$ ./tipc-pipe -l --sock_type SOCK_DGRAM
```

{{< /columns >}}


## 6.1. Connection Oriented Messaging
{{< columns >}}
### Client
```c++
sock_type = <SOCK_STREAM or SOCK_SEQPACKET>;
sd = socket(AF_TIPC, sock_type, 0);




connect(sd, &server_addr, sizeof(server_addr));

write(sd, buf, len);
recvfrom(sd, buf, buf_size, 0, &peer, &addr_size);

close(sd);
```

```sh
$ cd tipcutils/demos/tipc-pipe
$ tipc-pipe --sock_type SOCK_STREAM
```

<--->
### Server
```c++
sock_type = <SOCK_STREAM or SOCK_SEQPACKET>;

sd = socket(AF_TIPC, socket_type, 0);
bind(sd, &server, sizeof(server));
listen(sd, 0);

peer_sd = accept(sd, 0, 0);

recvfrom(peer_sd, buf, buf_size, 0, &peer, &addr_size);
write(peer_sd, buf, len);

close(peer_sd);

close(sd);
```

```sh
$ cd tipcutils/demos/tipc-pipe
$ tipc-pipe -l --sock_type SOCK_STREAM
```

{{< /columns >}}



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
