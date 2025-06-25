---
title: "Reproduce TIPC Socket EAGAIN Error"
subtitle: "*Fully fill the socket send buffer*"
---

When we send data with the send or sendmsg functions, it first goes into a buffer before being sent over the network. The buffer has a fixed-size.

What happens if the buffer is full?

- **Blocking mode**: The send function pauses and waits until there is space in the buffer. This space becomes available when the receiver reads some of the data.
- **Non-blocking mode**: The send function returns an EAGAIN error, which means the buffer is full and cannot accept more data right now.

In the following test, we will reproduce the EAGAIN error using a TIPC socket with a client and a server. 

- The client will continuously send a large amount of data to the server. 
- However, the server does not read any data from the socket.
- After some time, the send buffer of client becomes full, 
- As a result, the send function will wait if the socket is in blocking mode, or return an EAGAIN error if in non-blocking mode.


# 1. Blocking Mode
- server.c
```c
  
#include <linux/tipc.h>
#include <netinet/in.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char** argv)
{
    struct sockaddr_tipc server_addr = {
        .family = AF_TIPC,
        .addrtype = TIPC_ADDR_NAMESEQ,
        .scope = TIPC_ZONE_SCOPE,
        .addr.nameseq = {
            .type = 18888,
            .lower = 17,
            .upper = 17
        }
    };
 
    int listenfd = socket(AF_TIPC, SOCK_STREAM, 0);
 
    bind(listenfd, &server_addr, sizeof(server_addr));
 
    listen(listenfd, 0);
 
    int peerfd = accept(listenfd, 0, 0);
 
    char buf[2048];
    int total_received = 0;
 
    while (1)
    {
        printf("press a key to call recv()\n");
        getc(stdin);
 
        int n = recv(peerfd, buf, 2048, 0);
        printf("recv returns: %d\n", n);
 
        if (n > 0)
        {
            total_received += n;
            printf("total received: %d\n", total_received);
        }
        else
        {
            printf("recv got errno: %d\n", errno);
        }
    }
}
  
```

- client.c
```c
  
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <netinet/in.h>
#include <linux/tipc.h>

int main(int argc, char** argv)
{
    struct sockaddr_tipc server_addr = {
        .family = AF_TIPC,
        .addrtype = TIPC_ADDR_NAME,
        .addr.name = {
            .name = {
                .type = 18888,
                .instance = 17
            },
            .domain = 0
        }
    };
 
    int sockfd = socket(AF_TIPC, SOCK_STREAM, 0);

    connect(sockfd, &server_addr, sizeof(server_addr));
 
    char buf[2048];
    memset(buf, 'A', 2048);
   
    int total_sent = 0;
 
    while (1)
    {
        printf("sending...\n");
        int n = send(sockfd, buf, 2048, 0);
        printf("send return: %d\n", n);
 
        if (n > 0)
        {
            total_sent += n;
            printf("total sent: %d\n", total_sent);
            sleep(1);
        }
        else
        {
            printf("send got errno: %d\n", errno);
            sleep(5);
        }
    }
}
  
```

- Build.
```sh
  
$ gcc -O0 -g -Wno-incompatible-pointer-types client.c -o client
$ gcc -O0 -g -Wno-incompatible-pointer-types server.c -o server
  
```

- Load TIPC module.
```sh
  
$ modprobe tipc
$ lsmod | grep tipc
  
```

- Run
```sh
  
$ ./server
$ ./client
  
```

- After send buffer of socket reaches 88064 bytes, it becomes full, and the client pauses at send().
```sh
  
# client

sending...
send return: 2048
total sent: 2048
sending...
send return: 2048
total sent: 4096
.....
sending...
send return: 2048
total sent: 86016
sending...
send return: 2048
total sent: 88064
sending...
  
```

- On the server terminal, press a key to call recv and consume data from the socket buffer. After some time, once space becomes available in the buffer, the client will be able to continue sending messages.
```sh
  
# server  

press a key to call recv()

recv returns: 2048
total received: 2048
press a key to call recv()

...

recv returns: 2048
total received: 22528
press a key to call recv()
  
```


```sh
  
# client

sending...
send return: 2048
total sent: 88064
sending...

send return: 2048
total sent: 90112
sending...
send return: 2048
total sent: 92160
sending...
send return: 2048
total sent: 94208
  
```


# 2. Non-blocking Mode
- client_nonblocking.c
```c
  
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <netinet/in.h>
#include <linux/tipc.h>

int main(int argc, char** argv)
{
    struct sockaddr_tipc server_addr = {
        .family = AF_TIPC,
        .addrtype = TIPC_ADDR_NAME,
        .addr.name = {
            .name = {
                .type = 18888,
                .instance = 17
            },
            .domain = 0
        }
    };

    int sockfd = socket(AF_TIPC, SOCK_STREAM, 0);

    // enable non-blocking
    int flags = fcntl(sockfd, F_GETFL, 0);
    fcntl(sockfd, F_SETFL, flags|O_NONBLOCK);

    connect(sockfd, &server_addr, sizeof(server_addr));

    char buf[2048];
    memset(buf, 'A', 2048);

    int total_sent = 0;

    while (1)
    {
        printf("sending...\n");
        int n = send(sockfd, buf, 2048, 0);
        printf("send return: %d\n", n);

        if (n > 0)
        {
            total_sent += n;
            printf("total sent: %d\n", total_sent);
            sleep(1);
        }
        else
        {
            printf("send got errno: %d\n", errno);
            sleep(5);
        }
    }
}
  
```

- Build
```sh
  
$ gcc -O0 -g -Wno-incompatible-pointer-types client_nonblocking.c -o client_nonblocking
  
```

- Run
```sh
  
$ ./server
$ ./client
  
```

- After send buffer of socket reaches 88064 bytes, it becomes full, and the send() function returns EAGAIN error.
```sh
  
# client

sending...
send return: 2048
total sent: 83968
sending...
send return: 2048
total sent: 86016
sending...
send return: 2048
total sent: 88064
sending...
send return: -1
send got errno: 11
sending...
send return: -1
send got errno: 11
sending...
send return: -1
send got errno: 11
  
```

- On the server terminal, press a key to call recv and consume data from the socket buffer. After some time, once space becomes available in the buffer, the client will be able to continue sending messages.
```sh
  
# server
recv returns: 2048
total received: 22528
press a key to call recv()

recv returns: 2048
total received: 24576
press a key to call recv()

recv returns: 2048
total received: 26624
press a key to call recv()
  
```

```sh
  
# client
sending...
send return: -1
send got errno: 11
sending...
send return: 2048
total sent: 90112
sending...
send return: 2048
total sent: 92160
sending...
send return: 2048
total sent: 94208
  
```


# 3. Non-blocking & Epoll
- client_epoll.c
```c
  
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <netinet/in.h>
#include <linux/tipc.h>

int main(int argc, char** argv)
{
    struct sockaddr_tipc server_addr = {
        .family = AF_TIPC,
        .addrtype = TIPC_ADDR_NAME,
        .addr.name = {
            .name = {
                .type = 18888,
                .instance = 17
            },
            .domain = 0
        }
    };

    int sockfd = socket(AF_TIPC, SOCK_STREAM, 0);

    // enable non-blocking
    int flags = fcntl(sockfd, F_GETFL, 0);
    fcntl(sockfd, F_SETFL, flags|O_NONBLOCK);

    connect(sockfd, &server_addr, sizeof(server_addr));

    // edge trigger write event
    int epollfd = epoll_create1(0);
    struct epoll_event ev = {
        .events = EPOLLOUT | EPOLLET,
        .data.fd = sockfd
    };
    epoll_ctl(epollfd, EPOLL_CTL_ADD, sockfd, &ev);

    char buf[2048];
    memset(buf, 'A', 2048);

    int total_sent = 0;

    while (1)
    {
        printf("sending...\n");
        int n = send(sockfd, buf, 2048, 0);
        printf("send return: %d\n", n);

        if (n > 0)
        {
            total_sent += n;
            printf("total sent: %d\n", total_sent);
            sleep(1);
        }
        else
        {
            printf("send got errno: %d\n", errno);

            // if got EAGAIN, wait for EPOLLOUT (ready to write)
            if (errno == EAGAIN)
            {
                struct epoll_event events[4];
                printf("epoll_wait...\n");
                int nfds = epoll_wait(epollfd, events, 4, -1);
                printf("wake up: nfds=%d\n", nfds);

                for(int i = 0; i < nfds; i++)
                {
                    printf("epoll_event: events=0x%x, data.fd=%d\n",
                        events[i].events, events[i].data.fd);
                }
            }

            sleep(5);
        }
    }
}
  
```

```sh
  
$ gcc -O0 -g -Wno-incompatible-pointer-types client_epoll.c -o client_epoll
  
```

```sh
  
$ ./server
$ ./client_epoll
  
```

- After send buffer of socket reaches 88064 bytes, it becomes full, and the send() function returns EAGAIN error. Client pauses by epoll_wait, and wait for EPOLLOUT to wake up.
```sh
  
# client

send return: 2048
total sent: 88064
sending...
send return: -1
send got errno: 11
epoll_wait...
  
```

- On the server terminal, press a key to call recv and consume data from the socket buffer. After some time, once space becomes available in the buffer, the client will be waked up by EPOLLOUT event.
```sh
  
# server

recv returns: 2048
total received: 20480
press a key to call recv()

recv returns: 2048
total received: 22528
press a key to call recv()
  
```

```sh
  
# client

sending...
send return: 2048
total sent: 88064
sending...
send return: -1
send got errno: 11
epoll_wait...
wake up: nfds=1
epoll_event: events=0x4, data.fd=3
sending...
send return: 2048
total sent: 90112
  
```