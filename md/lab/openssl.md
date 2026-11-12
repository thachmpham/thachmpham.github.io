---
title: 'OpenSSL'
subtitle: '(Lab Setup Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column width=50%}

A lab to practice OpenSSL wth 2 docker containers. The h1 container acts as openssl server, h2 as openssl client.


Step 1: Setup keys and certificate on server.

- Create a private key.
- Create a certificate signing request from the priviate key.
- Sign the request to get a certificate.

Step 2: Setup trust store on client.

- Copy server's certificate to it's local.
- Add the certificate to its trust store database.

Purpose of the trust store:

- During tls handshake, server sends its certificate to client.
- Client validate server's certificate against its trust store.

Step 3: Connect the server and client by openssl s_server and s_client.


:::
::: {.column width=50%}

| Term | Meaning |
| :--- | :--- |
| Private Key | A hash string. Keep in private. To verify public key. |
| Public Key | Generated from private key. Share to public. |
| Certificate Signing Request| Generated from priviate key. Send to Certificate Authority to get a certificate. |
| Certificate | Binds an domain name or IP to a public key. |
| Trust Store | A collection of trusted certificates. |

:::
::::::::::::::


# Setup
## Nodes
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


## Keys, Certificates for Server
- Generate keys and certificates.
```sh
# create private key
h1$ openssl genpkey -algorithm RSA -out h1.pkey

# create certificate signing request
h1$ openssl req -new -key h1.pkey -out h1.req

# create certificate
h1$ openssl x509 -req -in h1.req -key h1.pkey -out h1.crt
```


## Trust Store for Client
- Copy server's certificate to client.
```sh
host$ docker cp h1:/root/h1.crt .
host$ docker cp h1.crt h2:/root
```

- Add server's certificate to client trust store.
```sh
# convert server's certificate to PEM format
h2$ openssl x509 -in h1.crt -out h1.pem -outform PEM

# add server's certificate to trust store
h2$ cp h1.pem /etc/pki/ca-trust/extracted/anchors
h2$ update-ca-trust
```


## Keys, Certificates for Client
- Generate keys and certificates.
```sh
# create private key
h2$ openssl genpkey -algorithm RSA -out h2.pkey

# create certificate signing request
h2$ openssl req -new -key h2.pkey -out h2.req

# create certificate
h2$ openssl x509 -req -in h2.req -key h2.pkey -out h2.crt
```


## Connect Server and Client
- Start openssl server.
```sh
h1$ openssl s_server -accept 172.19.0.2:4433 -cert h1.crt -key h1.pkey
```

- Start openssl client.
```sh
h2$ openssl s_client -connect 172.19.0.2:4433 -cert h2.crt -key h2.pkey -verify_return_error -CAfile h1.pem
```


# References
- [Add Trust Store on Fedora](https://ajmaradiaga.com/Adding-trusting-CA-Fedora/)
- [CA Certificates Trust Store](https://www.golinuxcloud.com/linux-ca-certificates-trust-store/)
- [Self-Signed Certificates](https://learn.microsoft.com/en-us/azure/application-gateway/self-signed-certificates)

