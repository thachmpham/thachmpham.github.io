---
title: 'OpenSSL: Client + Server'
subtitle: '(Lab Setup Series)'
---


# Overview

:::::::::::::: {.columns}
::: {.column width=50%}

A lab to practice openssl. The lab has 2 docker containers; one acts as server, one as client.

Step 1: Setup keys and certificate on server.

- Create a private key.
- Create a certificate signing request from the priviate key.
- Sign the request to get a certificate.

Step 2: Setup keys and certificate on client.

- Create a private key.
- Create a certificate signing request from the priviate key.
- Sign the request to get a certificate.

Step 3: Setup trust store on client.

- Copy server's certificate to client's trust store.

Step 4: Connect the server and client by openssl s_server and s_client.


:::
::: {.column width=50%}

| Term | Meaning |
| :--- | :------ |
| Private Key | A hash string. Keep in private. |
| Public Key | A hash string. Share to public. |
| CSR | Certificate Signing Request. Send to Certificate Authority to request a certificate. |
| Certificate | Digital identify of a server, website. |
| Trust Store | A collection of trusted certificates. During TLS handshake, server sends its certificate to client. Client validates server's certificate against client's trust store.|

:::
::::::::::::::


# Setup
## Create Nodes

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Setup containers.
```sh
host$ git clone https://github.com/thachmpham/lab.git

host$ cd lab/openssl

host$ docker compose build
host$ docker compose up --detach
```

:::
::: {.column width=50%}

- Step 2: To access server.
```sh
host$ docker exec -it h1 bash
```

- Step 3: To access client.
```sh
host$ docker exec -it h2 bash
```

:::
::::::::::::::


## Keys, Certificates

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Generate keys and certificates for servers.
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

:::
::: {.column width=50%}

- Step 2: Generate keys and certificates for client.
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

:::
::::::::::::::


## Trust Store
- Step 1: Copy server's certificate to client.
```sh
h2$ mkdir /root/trust-store

host$ docker cp h1:/root/h1.crt .
host$ docker cp h1.crt h2:/root/trust-store
```

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 2: Setup trust store directory on client.
```sh
h2$ openssl rehash /root/trust

h2$ s -lh /root/trust/
lrwxrwxrwx 1 root root    6 Nov 14 00:07 9741086f.0 -> h1.crt
-rw-r--r-- 1 root root 1.2K Nov 14 00:06 h1.crt
```

:::
::: {.column width=50%}

- Step 3: Verify server's certificate against client's trust store.
```sh
h2$ openssl verify -CApath /root/trust/ h1.crt
h1.crt: OK
```

:::
::::::::::::::


## Establish Connection

:::::::::::::: {.columns}
::: {.column width=50%}

- Step 1: Start openssl server on h1.
```sh
h1$ openssl s_server -accept 172.19.0.2:4433 -cert h1.crt -key h1.pkey
```

:::
::: {.column width=50%}

- Step 2: Start openssl client on h2.
```sh
h2$ openssl s_client -connect 172.19.0.2:4433 -cert h2.crt -key h2.pkey \
    -CApath /root/trust -verify_return_error
```

:::
::::::::::::::


# References
- [OpenSSL Lookup Hash Directory](https://docs.openssl.org/3.4/man3/X509_LOOKUP_hash_dir/#file-method)
- [Add Trust Store on Fedora](https://ajmaradiaga.com/Adding-trusting-CA-Fedora/)
- [CA Certificates Trust Store](https://www.golinuxcloud.com/linux-ca-certificates-trust-store/)
