---
title: 'Diameter'
subtitle: '(Lab Setup Series)'
---


# Overview
A lab to practice diameter protocol.


# Build
```sh
h1$ rpm -ql erlang-examples
/usr/lib64/erlang/lib/diameter-2.3.2.2/examples/code/GNUmakefile
/usr/lib64/erlang/lib/diameter-2.3.2.2/examples/code/client.erl
/usr/lib64/erlang/lib/diameter-2.3.2.2/examples/code/server.erl
```

```sh
h1$ cd /usr/lib64/erlang/lib/diameter-2.3.2.2/examples/code/
h1$ make
```

# Connect

- Start server.
```sh
h1$ cd /usr/lib64/erlang/lib/diameter-2.3.2.2/examples/code/

h1$ erl -pa .
> application:start(diameter).
> server:start().
> server:listen(tcp).
```

- Start client.
```sh
h2$ cd /usr/lib64/erlang/lib/diameter-2.3.2.2/examples/code/

h1$ erl -pa .
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

- Stop client.
```sh
> client:stop().
```

# References
- [RFC 6733](https://datatracker.ietf.org/doc/html/rfc6733).
- [Erlang Diameter API](https://www.erlang.org/doc/apps/diameter/api-reference.html).
- [Erlang Diameter Example Github](https://github.com/erlang/otp/tree/master/lib/diameter/examples).
- [Diameter New Generation AAA Protocol, Design, Practice and Application.](https://www.wiley.com/en-us/Diameter%3A+New+Generation+AAA+Protocol+-+Design%2C+Practice%2C+and+Applications-p-9781118875834).
