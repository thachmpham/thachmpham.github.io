---
title: "rsyslog"
---

# Build & Run
- Install dependencies.
```sh
$ apt install -y build-essential pkg-config libestr-dev libfastjson-dev zlib1g-dev \
uuid-dev libgcrypt20-dev liblogging-stdlog-dev libhiredis-dev \
uuid-dev liblogging-stdlog-dev flex bison libcurl4-openssl-dev
```

- Download source code.
```sh
$ wget http://www.rsyslog.com/download/files/download/rsyslog/rsyslog-8.2110.0.tar.gz
$ tar xf rsyslog-8.2110.0.tar.gz -C rsyslog
```

- Build.
```sh
$ cd rsyslog
$ ./configure --enable-debug
$ make
```

- Run without install.
```sh
$ rsyslog/tools/rsyslogd
```

- Or install and run.
```sh
$ make install
$ /usr/local/sbin/rsyslogd
```