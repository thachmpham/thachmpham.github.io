---
title:  'LDAP'
---


# 1. Introduction
LDAP (Lightweight Directory Access Protocol) is a protocol used to store, organize, and access directory information in a structured way. It is commonly used for user authentication, access control, and identity management in networks, allowing multiple applications and systems to share a centralized directory of users and resources.

OpenLDAP is an open-source implementation of the LDAP. OpenLDAP is widely used in enterprise environments to manage identities and enforce security policies efficiently.


# 2. Lab
We will setup a ldap server using OpenLDAP and send some requests to the server.

## 2.1. Install
```sh
  
$ apt install slapd ldap-utils
  
```


## 2.2. Slapd Server
Create file /etc/ldap/slapd.conf.
```sh
  
include     /etc/ldap/schema/core.schema
include     /etc/ldap/schema/cosine.schema
include     /etc/ldap/schema/inetorgperson.schema

pidfile     /var/run/slapd/slapd.pid
argsfile    /var/run/slapd/slapd.args
loglevel    none

modulepath  /usr/lib/ldap
moduleload  back_mdb

database     mdb
suffix      "dc=example,dc=com"
rootdn      "cn=Manager,dc=example,dc=com"
rootpw      fakepass
directory   /var/lib/ldap

access      to attrs=userPassword
            by anonymous auth
            by self write
            by * none

access      to *
            by self write
            by * none
  
```

Validate configuration.
```sh
  
$ slaptest -v -f /etc/ldap/slapd.conf
  
```

Start slapd server.
```sh
  
$ slapd -f /etc/ldap/slapd.conf -h ldap://127.0.0.1:12345
  
```

## 2.3. LDAP Requests
### 2.3.1. ldapadd
Create file data.ldif
```python
  
dn:             dc=example,dc=com
dc:             example
o:              example.com
objectClass:    top
objectClass:    dcObject
objectClass:    organization


dn:             ou=users,dc=example,dc=com
ou:             users
objectClass:    organizationalUnit


dn:             uid=alice,ou=users,dc=example,dc=com
ou:             users
uid:            alice
sn:             jane
cn:             alex jane
objectClass:    person
objectClass:    organizationalPerson
objectClass:    inetorgperson
  
```

Validate data.
```sh
    
$ slapadd -v -u -c -f /etc/ldap/slapd.conf -l data.ldif
  
```

Send ldapadd request.
```sh
  
$ ldapadd -x -H ldap://127.0.0.1:12345 -w fakepass -D 'cn=Manager,dc=example,dc=com' -f data.ldif
  
```


### 2.3.2. ldapsearch
```sh
  
$ ldapsearch -x -H ldap://127.0.0.1:12345 -w fakepass -D 'cn=Manager,dc=example,dc=com' -b 'dc=example,dc=com'
$ ldapsearch -x -H ldap://127.0.0.1:12345 -w fakepass -D 'cn=Manager,dc=example,dc=com' -b 'ou=users,dc=example,dc=com'
$ ldapsearch -x -H ldap://127.0.0.1:12345 -w fakepass -D 'cn=Manager,dc=example,dc=com' -b 'uid=alice,ou=users,dc=example,dc=com'
  
```

### 2.3.3. ldapdelete
```sh
  
$ ldapdelete -x -H ldap://127.0.0.1:12345 -w fakepass -D 'cn=Manager,dc=example,dc=com' 'uid=alice,ou=users,dc=example,dc=com'
  
```


# 3. Troubleshooting
### 3.1. Dump Database
```sh
  
$ slapcat
  
```


### 3.2. Enable debug
```sh
  
$ slapd -d 1 -f /etc/ldap/slapd.conf -h ldap://127.0.0.1:389
  
```


### 3.3. Permission denied when open slapd.conf
Problem: The below error message is shown when start slapd server.
```sh
  
Could not open config file “slapd.conf”: Permission denied.
  
```

Reason: AppArmor prevents slapd to open file from unconfigured folders.
```sh
  
$ dmesg
  
```

```sh
  
audit: type=1400 apparmor="DENIED" operation="open" class="file" profile="/usr/sbin/slapd" 
name="/root/ldap/slapd.conf" pid=30197 comm="slapd" requested_mask="r" denied_mask="r"
  
```

Solution: Put slapd.conf in folder /etc/ldap.


# References
- [openldap.org](https://www.openldap.org/)

