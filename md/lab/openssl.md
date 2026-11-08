
- Generate server pkey, certificates.
```sh
# create pkey
h1$ openssl genpkey -algorithm RSA -out h1.pkey

# create certificate signing request
h1$ openssl req -new -key h1.pkey -out h1.req

# create certificate
h1$ openssl x509 -req -in h1.req -key h1.pkey -out h1.crt
```


- Copy server certificate to client.
```sh
host$ docker cp h1:/root/h1.crt .
host$ docker cp h1.crt h2:/root
```

- Add server certificate to client trust store.
```sh
h2$ cp h1.crt /etc/pki/ca-trust/extracted/
h2$ update-ca-trust
h2$ trust list
```


- https://www.golinuxcloud.com/linux-ca-certificates-trust-store/
- https://learn.microsoft.com/en-us/azure/application-gateway/self-signed-certificates