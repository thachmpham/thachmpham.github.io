---
title: "LD_PRELOAD"
subtitle: "*Make a Shared Library Do Something Different*"
---

**`LD_PRELOAD`** is an environment variable that allows to load a shared library before any other libraries when a program starts.
This gives us the ability to override standard functions with custom implementations without modifiying the original program.

**`LD_DEBUG`** is an environment varibale used to trace and debug the dynamic linker during program startup.
```sh
  
LD_PRELOAD="lib_01.so" program
LD_PRELOAD="lib_01.so:lib_02.so" program

LD_DEBUG=all LD_PRELOAD="lib_01.so:lib_02.so" program

man ld.so
  
```

With LD_PRELOAD, we can monitor standard library function calls by add tracing or simulate rare errors - like making a socket call to return EAGAIN.


# 1. Override malloc
In this section, we will override the malloc function. We print a message each time malloc is called.

- File main.c
```c
  
#include <stdlib.h>

int main(int argc, char** argv)
{
    char *p = malloc(8);
    free(p);

    return 0;
}
  
```

- Build
```sh
  
$ gcc -o main main.c
  
```

- File mymalloc.c
```c
  
#define _GNU_SOURCE

#include <stdio.h>
#include <dlfcn.h>

static void* (*real_malloc)(size_t) = NULL;

void* malloc(size_t size)
{
    if(real_malloc == NULL)
    {
        real_malloc = dlsym(RTLD_NEXT, "malloc");

        if (NULL == real_malloc)
        {
            fprintf(stderr, "dlsym failed: %s\n", dlerror());
        }
    }

    fprintf(stderr, "malloc(%ld) = ", size);
    void *p = real_malloc(size);
    fprintf(stderr, "%p\n", p);
    return p;
}
  
```

- Build
```sh
  
$ gcc -o mymalloc.so -fPIC -shared mymalloc.c -ldl
  
```

- Run program normally.
```sh
  
$ ./main
  
```

- Run with preload library.
```sh
  
$ LD_PRELOAD=./mymalloc.so ./main

malloc(8) = 0x593c4106e2a0
  
```

- Enable linker log.
```sh
  
$ LD_DEBUG=all LD_PRELOAD=./mymalloc.so ./main

file=./mymalloc.so [0];  needed by ./main [0]
file=./mymalloc.so [0];  generating link map
  dynamic: 0x00007272bc74cdf8  base: 0x00007272bc749000   size: 0x0000000000004030
    entry: 0x00007272bc749000  phdr: 0x00007272bc749040  phnum:                 11

file=libc.so.6 [0];  needed by ./main [0]
file=libc.so.6 [0];  generating link map
  dynamic: 0x00007272bc602940  base: 0x00007272bc400000   size: 0x0000000000211d90
    entry: 0x00007272bc42a390  phdr: 0x00007272bc400040  phnum:                 14

relocation processing: /lib/x86_64-linux-gnu/libc.so.6
symbol=malloc;  lookup in file=./main [0]
symbol=malloc;  lookup in file=./mymalloc.so [0]
binding file /lib/x86_64-linux-gnu/libc.so.6 [0] to ./mymalloc.so [0]: normal symbol `malloc' [GLIBC_2.2.5]

relocation processing: ./mymalloc.so (lazy)
symbol=malloc;  lookup in file=./main [0]
symbol=malloc;  lookup in file=./mymalloc.so [0]
binding file ./main [0] to ./mymalloc.so [0]: normal symbol `malloc' [GLIBC_2.2.5]

calling init: /lib/x86_64-linux-gnu/libc.so.6
calling init: ./mymalloc.so

initialize program: ./main
transferring control: ./main

malloc(8) = 0x60827a8082a0
  
```


# 2. Override sendmsg
In this section we will override the sendmsg function to simulate socket error. We get the error code from environment variable.

- File server.py
```python
  
import socket

PORT = 12345
BUFFER_SIZE = 1024

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.bind(("0.0.0.0", PORT))

print(f"UDP server listening on port {PORT}...")

while True:
    data, addr = sock.recvfrom(BUFFER_SIZE)
    print(f"Received {len(data)} bytes from {addr}: {data.decode()}")
  
```

- File client.c
```c
  
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <errno.h>

int main()
{
    int sockfd;
    struct sockaddr_in dest_addr;
    const char *msg = "Hello via sendmsg";

    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd < 0)
    {
        perror("socket");
        return 1;
    }

    memset(&dest_addr, 0, sizeof(dest_addr));
    dest_addr.sin_family = AF_INET;
    dest_addr.sin_port = htons(12345);
    inet_pton(AF_INET, "127.0.0.1", &dest_addr.sin_addr);

    struct iovec iov;
    iov.iov_base = (void *)msg;
    iov.iov_len = strlen(msg);

    struct msghdr msgh;
    memset(&msgh, 0, sizeof(msgh));
    msgh.msg_name = &dest_addr;
    msgh.msg_namelen = sizeof(dest_addr);
    msgh.msg_iov = &iov;
    msgh.msg_iovlen = 1;

    ssize_t sent = sendmsg(sockfd, &msgh, 0);
    if (sent < 0)
    {
        fprintf(stderr, "Send failed, errno=%d: %s\n", errno, strerror(errno));
        close(sockfd);
        return 1;
    }

    printf("Sent %ld bytes\n", sent);
    close(sockfd);
    return 0;
}
  
```

- File mysendmsg.c
```c
  
#define _GNU_SOURCE

#include <stdio.h>
#include <sys/socket.h>
#include <dlfcn.h>
#include <errno.h>
#include <stdlib.h>

static ssize_t (*real_sendmsg)(int sockfd, const struct msghdr *msg, int flags) = NULL;

static int get_errno_from_env(void)
{
    const char *val = getenv("MY_ERRNO");
    return val ? atoi(val) : 0;
}

ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags)
{
    if (!real_sendmsg)
    {
        real_sendmsg = dlsym(RTLD_NEXT, "sendmsg");
        if (!real_sendmsg)
        {
            fprintf(stderr, "Error in dlsym(sendmsg): %s\n", dlerror());
            errno = EIO;
            return -1;
        }
    }

    int val = get_errno_from_env();
    if (val != 0)
    {
        fprintf(stderr, "sendmsg: MY_ERRNO=%d\n", val);
        errno = val;
        return -1;
    }

    return real_sendmsg(sockfd, msg, flags);
}
  
```

- Build.
```sh
  
$ gcc -o client client.c
$ gcc -shared -fPIC -o mysendmsg.so mysendmsg.c -ldl
  
```

- Run server.
```sh
  
$ python3 server.py
  
```

- Run client normally.
```sh
  
$ ./client

Sent 17 bytes
  
```

- Simulate socket error ECONNABORTED (errno=103).
```sh
  
$ MY_ERRNO=103 LD_PRELOAD=./mysendmsg.so ./client

sendmsg: MY_ERRNO=103
Send failed, errno=103: Software caused connection abort
  
```


# Reference
- [man ld.so](https://man7.org/linux/man-pages/man8/ld.so.8.html)