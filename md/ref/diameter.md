---
title: "Diameter Protocol"
---


## Theory

* * * * * 

:::::::::::::: {.columns}
::: {.column width=30%}

Diameter is an AAA protocol, mainly used in telecom.

Authentication

- Verify the user identity: username/password, certificate.
- Output: accept, reject.

Authorization

- Determine permissions for user.
- Output: permissions, access rules.

Accounting

- Track resource usages for billing: call duration, data volume.
- Output: charging records.

:::
::: {.column width=30%}

Diameter follows two-layer structure.

Base Protocol Layer - Infrastructure.

- Peer discovery & capability negotiation (CER, CEA).
- Connection management and keepalive (DWR, DWA).
- Message routing (realm, host, application-id).

Application Layer - Business Logic.

- Online charing (CCR, CCA).
- Offline charing (ACR, ACA).
- Gx Policy control (RAR, RAA).

<br>

Message Format

A diameter message contains a header and a payload.  
The payload is a list of attribute–value pairs (AVPs).

:::
::: {.column width=30%}

A diameter node can act as a client, server or agent.

Client

- User device sends radio, IP signaling, not diameter.
- Diameter client node receives the signal and translate to diameter requests.
- Send diameter requests to server.

Server

- Handle diameter requests and reply answers.

Agent

- Forward messages.

<br>

Diameter uses TCP or SCTP as transport protocol.

:::
::::::::::::::

## Lab 01: Erlang Diameter 

* * * * *

:::::::::::::: {.columns}
::: {.column}

Step 1: Install erlang.
```sh
$ apt install -y erlang

$ dpkg -l | grep erlang
erlang
erlang-diameter
erlang-examples
```

:::
::: {.column}

Step 2: Build diameter example code.

```sh
$ dpkg -L erlang-examples
/usr/lib/erlang/lib/diameter-2.2.4/examples/code/server.erl
/usr/lib/erlang/lib/diameter-2.2.4/examples/code/client.erl

$ cd /usr/lib/erlang/lib/diameter-2.2.4/examples/code/

$ erlc *.erl
# build .erl file to .beam
```

:::
::::::::::::::

:::::::::::::: {.columns}
::: {.column}

Step 3: Start server.

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

:::
::: {.column}

Step 4: Start server.
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

:::
::::::::::::::

Step 5: Capture pcap.
```sh
$ tcpdump -i lo 'tcp port 3868' -w acr_aca.pcap

$ tshark -r acr_aca.pcap 'tcp.len > 0'
    1   0.000000    127.0.0.1 → 127.0.0.1    DIAMETER 222 cmd=Accounting Request(271) flags=RP-- appl=Diameter Common Messages(0) h2h=a497849b e2e=a497849b 
    3   0.000301    127.0.0.1 → 127.0.0.1    DIAMETER 214 cmd=Accounting Answer(271)  flags=-P-- appl=Diameter Common Messages(0) h2h=a497849b e2e=a497849b

$ tshark -r acr_aca.pcap -O diameter 'tcp.len > 0'
```
Sample: [acr_aca.pcap](https://github.com/thachmpham/samples/blob/main/pcap/diameter/acr_aca.pcap).

* * * * *


## References
- [Erlang Diameter API](https://www.erlang.org/doc/apps/diameter/api-reference.html).
- [RFC 6733](https://datatracker.ietf.org/doc/html/rfc6733).
- Diameter New Generation AAA Protocol, Design, Practice and Application. Hassen, Sebastien, Jean, Jouni.