---
weight: 1
title: "TIPC"
bookToc: false
bookFlatSection: true
---

# TIPC
# Table of Content
[1. Introduction](#1-introduction)  
[2. Setup a Cluster](#2-setup-a-cluster)  
[3. Hello World](#3-hello-world)  
[4. Addressing](#4-addressing)  
[5. Service & Topology Tracking](#5-service--topology-tracking)  
[6. Messaging](#6-messaging)  
[X. Troubleshooting](#x-troubleshooting)  


# 1. Introduction
Transparent Inter-Process Communication (TIPC) is a communication protocol used for inter-process communication (IPC) in distributed systems. TIPC supports unicast, anycast, multicast and broadcast datagram messaging. It also provides a group membership tracker service that ensuring consistency between membership events and message delivery.

Websites:
- [tipc.sourceforge.net](https://tipc.sourceforge.net)

Papers:
- [TIPC Communication Group Netdev 0x12 Paper](https://sourceforge.net/projects/tipc/files/TIPC%20Communication%20Groups%20Netdev%200x12%20Paper.pdf/download)

Implementation:
- [github.com/torvalds/linux/tree/master/net/tipc](https://github.com/torvalds/linux/tree/master/net/tipc)

Headers:
- /usr/include/linux/tipc.h

# 2. Setup a Cluster
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
## tipc_socket_addr 
```c++
struct tipc_socket_addr {
        __u32 ref;
        __u32 node;
};
```
Address type tipc_socket_addr denotes unicast address (socket-to-socket). It contains two identifiers, a port number and a node number, both given by the system at the moment the socket is created.
<--->

## tipc_service_addr
```c++
struct tipc_service_addr {
        __u32 type;
        __u32 instance;
};
```
The tipc_service_addr type represents anycast or multicast addresses with two numbers: a service type identifier and an instance identifier, set by the application programmer. Multiple TIPC sockets can bind to the same service address. Anycast delivers a message to any socket bound to the address, while multicast sends a copy to all sockets bound to it.
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

## 4.2. Samples

{{< tabs "address_samples" >}}

{{< tab "tipc_service_addr" >}}

Bind a TIPC service with type is 100 and instance ID is 1. Full code at [HERE](https://github.com/thachmpham/samples/blob/main/tipc/tipc_service_addr.c).

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
$ ./demo
```

```sh
$ tipc nametable show
Type       Lower      Upper      Scope    Port       Node
100        1          1          cluster  169009847  000c29a1859d
100        1          1          cluster  2888149149 000c29a1859d
```

- Two sockets were bound to the same service address {type=100, instance=1}.
{{< /tab >}}

{{< tab "tipc_service_range" >}}

Bind a TIPC service with type is 100 and range is 0-10. Full code at [HERE](https://github.com/thachmpham/samples/blob/main/tipc/tipc_service_range.c).
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
Service tracking means that a user can create a socket, connect it to a built-in topology server and subscribe for events related to a specified service address or address range.

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

## 5.2. Sample topology_subscr_demo
Full code at [topology_subscr_demo](https://github.com/TIPC/tipcutils/tree/master/demos/topology_subscr_demo).
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

```sh
$ cd tipcutils/demos/topology_subscr_demo
$ make
$ ./top_client
$ ./top_server
```

```sh
# capture tipc packets
# 0x88ca is ethertype of TIPC. Defined in /usr/include/linux/if_ether.h
$ tshark -i ens160 -f 'ether proto 0x88ca' -w sample.pcap


# display tipc packets, except Link State Maintenance messages
# because there too many messages of this type
# 7 is the ID of Link State Maintenance messages
$ tshark -r sample.pcap -Y 'tipc.usr != 7'
81      7.925556593     90.1698.2539    → 0.0.0            TIPC 74 Name Dist    Publication type:18888 inst:17
128     12.607446355    133.2322.2465   → 90.1698.2539     TIPC 69 Payld:Low    NamedMsg type:18888 inst:17
129     12.609085805    90.1698.2539    → 133.2322.2465    TIPC 60 Payld:Low    DirectMsg
130     12.609085930    90.1698.2539    → 0.0.0            TIPC 74 Name Dist    Withdrawal type:18888 inst:17
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


## 6.2. Connection Oriented Messaging
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


## 6.3. Group Messaging
## 6.3.1. Broadcast
{{< columns>}}
### Client
```c++
// create tipc socket
int socket_fd = socket(AF_TIPC, SOCK_RDM, 0);

// requests to join group
struct tipc_group_req request;
request.type = 4711;
request.instance = 0;
request.scope = TIPC_NODE_SCOPE;
setsockopt(socket_fd, SOL_TIPC, TIPC_GROUP_JOIN, &request, sizeof(request));

// send broadcast message
char buf[32] = "hello";
int ret = send(socket_fd, buf, strlen(buf)+1, 0);
```
[broadcast_client.c](https://github.com/thachmpham/samples/blob/main/tipc/broadcast_client.c)
<--->

### Server
```c++
// create tipc socket
int sockfd = socket(AF_TIPC, SOCK_RDM, 0);

// requests to join group
struct tipc_group_req request;
request.type = 4711;
request.instance = 0;
request.scope = TIPC_NODE_SCOPE; // or TIPC_CLUSTER_SCOPE
setsockopt(sockfd, SOL_TIPC, TIPC_GROUP_JOIN, &request, sizeof(request));

// wait for messages
struct msghdr msg;
recvmsg(sockfd, &msg, 0);
```
[broadcast_server.c](https://github.com/thachmpham/samples/blob/main/tipc/broadcast_server.c)

{{< /columns>}}

## 6.3.2. Multicast
{{< columns>}}
### Client
```c++
struct sockaddr_tipc server_addr;
server_addr.family = AF_TIPC;
server_addr.addrtype = TIPC_ADDR_MCAST;
server_addr.addr.nameseq.type = 18888;
server_addr.addr.nameseq.lower = 0;
server_addr.addr.nameseq.upper = 399;

sd = socket(AF_TIPC, SOCK_RDM, 0);

sendto(sd, buf, strlen(buf) + 1, 0, 
          &server_addr, sizeof(server_addr));
```
[client_tipc.c](https://github.com/TIPC/tipcutils/blob/master/demos/multicast_demo/client_tipc.c)
<--->

### Server
```c++
struct sockaddr_tipc server_addr;
server_addr.family = AF_TIPC;
server_addr.addrtype = TIPC_SERVICE_RANGE;
server_addr.scope = TIPC_CLUSTER_SCOPE;
server_addr.addr.nameseq.type = 18888;
server_addr.addr.nameseq.lower = 0;
server_addr.addr.nameseq.upper = 399;

sd = socket (AF_TIPC, SOCK_RDM, 0);
bind(sd, &server_addr, sizeof(server_addr));

recvfrom(sd, buf, buf_size, 0, _, _);
```
[server_tipc.c](https://github.com/TIPC/tipcutils/blob/master/demos/multicast_demo/server_tipc.c)
{{< /columns>}}

## 6.3.3. Anycast
```text
ssize_t send(int sockfd, const void *buff, size_t nbytes, int flags)
  This routine has two uses:
    1) Attempt to send a message from a connected socket to its peer socket.
    2) Attempt to send a group broadcast from a group member socket.

ssize_t sendmsg(int sockfd, struct msghdr *msg, int flags)
  Attempt to send a message from the socket to the specified destination. There are three cases:
    1) If the destination is a socket address the message is unicast to that specific socket.
    2) If the destination is a service address, it is an anycast to any matching destination.
    3) If the destination is a service range, the message is a multicast to all matching sockets.
    Note however that the rules for what is a match differ between datagram and group messaging.
```  

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
