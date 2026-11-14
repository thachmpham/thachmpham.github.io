---
title: 'OpenSSL: Client + Server'
subtitle: '(Lab Setup Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column width=50%}

A lab to practice openssl. The lab has 2 docker containers; one acts as server, the other one as client.


Step 1: Setup keys and certificate on server.

- Create a private key.
- Create a certificate signing request from the priviate key.
- Sign the request to get a certificate.

Step 2: Setup keys and certificate on client.

- Create a private key.
- Create a certificate signing request from the priviate key.
- Sign the request to get a certificate.

Step 3: Setup trust store on client.

- Copy server's certificate to it's local.
- Add the certificate to its trust store database.

Step 4: Connect the server and client by openssl s_server and s_client.


:::
::: {.column width=50%}

| Term | Meaning |
| :--- | :--- |
| Private Key | A hash string. Keep in private. |
| Public Key | A hash string. Share to public. |
| Certificate Signing Request | Send to Certificate Authority to request a certificate. |
| Certificate | Digital identify of a server, website. |
| Trust Store | A collection of trusted certificates. |

<br>

Trust store:

- During tls handshake, server sends its certificate to client.
- Client validate server's certificate against client's trust store.

:::
::::::::::::::


# Setup
## Create Nodes
- Setup containers.
```sh
host$ git clone https://github.com/thachmpham/lab.git
host$ cd lab/openssl
host$ docker compose build
host$ docker compose up --detach
```

- To access server.
```sh
host$ docker exec -it h1 bash
```

- To access client.
```sh
host$ docker exec -it h2 bash
```


## Keys, Certificates on Server
- Generate keys and certificates.
```sh
# create private key
h1$ openssl genpkey -algorithm RSA -out h1.pkey
h1$ openssl pkeygen -in h1.pkey -text -noout

# create certificate signing request
h1$ openssl req -new -key h1.pkey -out h1.req
h1$ openssl req -in h1.req -text -noout

# create certificate
h1$ openssl x509 -req -in h1.req -key h1.pkey -out h1.crt
h1$ openssl x509 -in h1.crt -text -noout
```


## Keys, Certificates on Client
- Generate keys and certificates.
```sh
# create private key
h2$ openssl genpkey -algorithm RSA -out h2.pkey
h1$ openssl pkeygen -in h2.pkey -text -noout

# create certificate signing request
h2$ openssl req -new -key h2.pkey -out h2.req
h1$ openssl req -in h2.req -text -noout

# create certificate
h2$ openssl x509 -req -in h2.req -key h2.pkey -out h2.crt
h1$ openssl x509 -in h2.crt -text -noout
```


## Trust Store on Client
- Copy server's certificate to client.
```sh
host$ docker cp h1:/root/h1.crt .
host$ docker cp h1.crt h2:/root
```

- Setup trust store directory.
```sh
h2$ mkdir /root/trust
h2$ mv h1.crt /root/trust

h2$ openssl rehash /root/trust

h2$ s -lh /root/trust/
lrwxrwxrwx 1 root root    6 Nov 14 00:07 9741086f.0 -> h1.crt
-rw-r--r-- 1 root root 1.2K Nov 14 00:06 h1.crt
```

- Verify server's certificate against trust store.
```sh
h2$ openssl verify -CApath /root/trust/ h1.crt
h1.crt: OK
```


## Establish Connection
- Start openssl server.
```sh
h1$ openssl s_server -accept 172.19.0.2:4433 -cert h1.crt -key h1.pkey
```

- Start openssl client.
```sh
h2$ openssl s_client -connect 172.19.0.2:4433 -cert h2.crt -key h2.pkey -CApath /root/trust -verify_return_error
SSL handshake has read 2467 bytes and written 1617 bytes
Verification: OK
```


# References
- [OpenSSL Lookup Hash Directory](https://docs.openssl.org/3.4/man3/X509_LOOKUP_hash_dir/#file-method)
- [Add Trust Store on Fedora](https://ajmaradiaga.com/Adding-trusting-CA-Fedora/)
- [CA Certificates Trust Store](https://www.golinuxcloud.com/linux-ca-certificates-trust-store/)
