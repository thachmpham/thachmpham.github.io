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

> client:stop().
```

:::
::::::::::::::

Step 5: Capture pcap.
```sh
# tcpdump -i lo -w hello.pcap 'tcp port 3868' --print
$ tshark -i lo -w hello.pcap -f "tcp port 3868" -P -l

$ tshark -r hello.pcap -Y 'diameter'
4   0.001318243  127.0.0.1 → 127.0.0.1  DIAMETER 190 cmd=Capabilities-Exchange Request(257) flags=R--- appl=Diameter Common Messages(0) h2h=acb06dc3 e2e=acb06dc3 | 
6   0.002841007  127.0.0.1 → 127.0.0.1  DIAMETER 202 cmd=Capabilities-Exchange Answer(257)  flags=---- appl=Diameter Common Messages(0) h2h=acb06dc3 e2e=acb06dc3 | 
8  28.409234033  127.0.0.1 → 127.0.0.1  DIAMETER 134 cmd=Device-Watchdog Request(280)       flags=R--- appl=Diameter Common Messages(0) h2h=acb06dc4 e2e=acb06dc4 | 
9  28.410384934  127.0.0.1 → 127.0.0.1  DIAMETER 146 cmd=Device-Watchdog Answer(280)        flags=---- appl=Diameter Common Messages(0) h2h=acb06dc4 e2e=acb06dc4 | 
11 34.159011885  127.0.0.1 → 127.0.0.1  DIAMETER 222 cmd=Accounting Request(271)            flags=RP-- appl=Diameter Common Messages(0) h2h=acb06dc5 e2e=acb06dc5 | 
12 34.161076648  127.0.0.1 → 127.0.0.1  DIAMETER 214 cmd=Accounting Answer(271)             flags=-P-- appl=Diameter Common Messages(0) h2h=acb06dc5 e2e=acb06dc5 | 
14 48.667674359  127.0.0.1 → 127.0.0.1  DIAMETER 146 cmd=Disconnect-Peer Request(282)       flags=R--- appl=Diameter Common Messages(0) h2h=acb06dc6 e2e=acb06dc6 | 
15 48.668208006  127.0.0.1 → 127.0.0.1  DIAMETER 146 cmd=Disconnect-Peer Answer(282)        flags=---- appl=Diameter Common Messages(0) h2h=acb06dc6 e2e=acb06dc6 |

$ tshark -r hello.pcap -O 'diameter' -Y diameter > hello.txt
```

Sample: [hello.pcap](https://github.com/thachmpham/samples/blob/main/pcap/diameter/hello.pcap), [hello.txt](https://github.com/thachmpham/samples/blob/main/pcap/diameter/hello.txt).

* * * * *


## References
- [Erlang Diameter API](https://www.erlang.org/doc/apps/diameter/api-reference.html).
- [RFC 6733](https://datatracker.ietf.org/doc/html/rfc6733).
- Diameter New Generation AAA Protocol, Design, Practice and Application. Hassen, Sebastien, Jean, Jouni.