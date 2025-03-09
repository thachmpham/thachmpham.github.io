---
title:  'LDAP'
---

# 1. Introduction
LDAP (Lightweight Directory Access Protocol) is a protocol used to store, organize, and access directory information in a structured way. It is commonly used for user authentication, access control, and identity management in networks, allowing multiple applications and systems to share a centralized directory of users and resources.

Some libraries implemented LDAP:

- OpenLDAP
- 389 Directory Server
- Go-LDAP
- Python-LDAP


# 2. OpenLDAP
## 2.1. Install
```sh
  
$ apt install slapd ldap-utils
  
```


## 2.2. Start slapd server
- Create file /etc/ldap/slapd.conf.
```sh
  
include /etc/ldap/schema/core.schema
include /etc/ldap/schema/cosine.schema
include /etc/ldap/schema/inetorgperson.schema

pidfile /var/run/slapd/slapd.pid
argsfile /var/run/slapd/slapd.args
loglevel none

modulepath /usr/lib/ldap
moduleload back_mdb

database mdb
suffix "dc=example,dc=com"
rootdn "cn=Manager,dc=example,dc=com"
rootpw 123456
directory /var/lib/ldap

access  to attrs=userPassword
        by anonymous auth
        by self write
        by * none

access  to *
        by self write
        by * none
  
```

- Verify config.
```sh
  
$ slaptest -v -f /etc/ldap/slapd.conf
  
```

- Start slapd server.
```sh
  
$ slapd -d 1 -f /etc/ldap/slapd.conf -h ldap://127.0.0.1:12345
  
```


## 2.3. Send Requests
- Send ldapsearch request.
```sh
  
$ ldapsearch -x -H ldap://127.0.0.1:12345 -W -D 'cn=Manager,dc=example,dc=com' -b "" -s base
  
```


## 2.X. Troubleshooting
### 2.X.1. Enable debug.
```sh
  
$ slapd -d 1 -f /etc/ldap/slapd.conf -h ldap://127.0.0.1:12345
  
```

### 2.X.2. Could not open config file "slapd.conf": Permission denied.
- AppArmor prevents slapd to open file from unconfigured folders.
```sh
  
$ dmesg
  
```

```sh
  
[37500.107883] audit: type=1400 audit(1741514962.790:396): apparmor="DENIED" operation="open" class="file" profile="/usr/sbin/slapd" name="/root/ldap/slapd.conf" pid=30197 comm="slapd" requested_mask="r" denied_mask="r" fsuid=0 ouid=0
  
```



