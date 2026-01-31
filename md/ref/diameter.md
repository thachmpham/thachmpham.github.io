---
title: "Diameter Protocol"
---


## Lab 01: Erlang OTP Diameter

* * * * *

Install erlang and explore diameter example code.
```sh
$ apt install -y erlang

$ dpkg -l | grep erlang
erlang
erlang-diameter
erlang-examples

$ dpkg -L erlang-examples
/usr/lib/erlang/lib/diameter-2.2.4/examples/code/server.erl
/usr/lib/erlang/lib/diameter-2.2.4/examples/code/client.erl
```

Build example.

```sh
$ cd /usr/lib/erlang/lib/diameter-2.2.4/examples/code/

$ erlc *.erl
# build .erl file to .beam
```

Server.
```sh
$ cd /usr/lib/erlang/lib/diameter-2.2.4/examples/code

$ erl -pa .
> application:start(diameter).
> server:start().
> server:listen(tcp).
# server listening at 127.0.0.1:3868
```

```sh
$ lsof -n -i :3868
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
beam.smp 11825 root   17u  IPv4  91515      0t0  TCP 127.0.0.1:3868 (LISTEN)
```

Client
```sh
$ cd /usr/lib/erlang/lib/diameter-2.2.4/examples/code

$ erl -pa .
> application:start(diameter).
> client:start().
> client:connect(tcp).
> client:call().

{ok,['ACA'|
     #{'Accounting-Record-Number' => 0,
       'Accounting-Record-Type' => 1,
       'Origin-Host' => <<"server.example.com">>,
       'Origin-Realm' => <<"example.com">>,'Result-Code' => 2001,
       'Session-Id' => <<"client;1831359964;1;nonode@nohost">>}]}
```

Capture pcap: [acr_aca.pcap](https://github.com/thachmpham/samples/blob/main/pcap/diameter/acr_aca.pcap).
```sh
$ tcpdump -i lo 'tcp port 3868' -w acr_aca.pcap

$ tshark -r acr_aca.pcap 'tcp.len > 0'
    1   0.000000    127.0.0.1 → 127.0.0.1    DIAMETER 222 cmd=Accounting Request(271) flags=RP-- appl=Diameter Common Messages(0) h2h=a497849b e2e=a497849b 
    3   0.000301    127.0.0.1 → 127.0.0.1    DIAMETER 214 cmd=Accounting Answer(271) flags=-P-- appl=Diameter Common Messages(0) h2h=a497849b e2e=a497849b

$ tshark -r acr_aca.pcap -O diameter 'tcp.len > 0'
```

* * * * *


## References
- [Erlang Diameter API](https://www.erlang.org/doc/apps/diameter/api-reference.html).
- [RFC 6733](https://datatracker.ietf.org/doc/html/rfc6733).
- Diameter: New Generation AAA Protocol, Design, Practice and Application, Hassen, Sebastien, Jean, Jouni.