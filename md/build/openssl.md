---
title: 'OpenSSL'
subtitle: '(Open Source Rebuild Series)'
---


# Overview
Build and run tests of the [OpenSSL library](https://openssl-library.org).

# Prepare Lab
- Clone code.
```sh
host$ git clone https://github.com/thachmpham/build.git
```


- Build container.
```sh
host$ cd build/openssl
host$ docker build --tag openssl .
```

- Create container.
```sh
host$ docker create --name h1 --hostname h1 openssl
host$ docker start h1
```

- Access container.
```sh
host$ docker exec -it h1 bash
```


# Demo: SSL Echo
- Start server.
```sh
h1$ cd openssl/demos/sslecho
h1$ ./sslecho s
```

- Start client.
```sh
h1$ ./sshecho c 127.0.0.1
```
