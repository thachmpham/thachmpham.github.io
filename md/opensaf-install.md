---
title:  'Build & Install OpenSAF'
---


# 1. Introduction.
- Create an Ubuntu Docker container.
- Inside the container:
    - Clone the OpenSAF source.
    - Build and install OpenSAF.
    - Start OpenSAF.


# 2. Setup Docker Container.
Create the docker container.
```sh
    
$ docker run --privileged -it -h SC-1 ubuntu:16.04
  
```

Inside the container, install packages needed for OpenSAF.
```sh
  
$ apt update

# packages for development
$ apt install -y mercurial gcc g++ libxml2-dev automake m4 autoconf libtool pkg-config make python-dev libsqlite3-dev rpm vim

# packages for runtime
$ apt install -y sqlite3 libxml2 psmisc
  
```


# 3. Build & Install OpenSAF.
Clone source OpenSAF.
```sh
  
$ hg clone http://hg.code.sf.net/p/opensaf/staging opensaf-staging
  
```

Build & Install.
```sh
  
$ cd opensaf-staging
$ ./bootstrap
$ ./configure --enable-tipc CPPFLAGS="-DRUNASROOT"
$ make
$ make install
$ ldconfig
    
```


# 4. Setup Configuration.

Generate `/etc/opensaf/imm.xml`.
```sh
  
$ cd /usr/local/share/opensaf/immxml/

# generate nodes.cfg
$ ./immxml-clustersize --sc-count 1
Successful, result stored in the file ./nodes.cfg

$ ./immxml-configure
Successfully generated the imm file: ./imm.xml.20241012_1044

$ cp imm.xml.20241012_1044 /etc/opensaf/imm.xml
  
```


Edit `/etc/opensaf/dtmd.conf`.
```sh
  
DTM_NODE_IP=172.17.0.2
# 172.17.0.2: IP of the eth0 interface.
  
```


Edit `/etc/opensaf/node_name`.
```sh
  
SC-1
# SC-1: hostname.
  
```


# 5. Start OpenSAF.
Start OpenSAF.
```sh
   
$ /etc/init.d/opensafd start
   
```

Check status.
```sh
  
$ /etc/init.d/opensafd status
safSISU=safSu=SC-1\,safSg=NoRed\,safApp=OpenSAF,safSi=NoRed1,safApp=OpenSAF
        saAmfSISUHAState=ACTIVE(1)
safSISU=safSu=SC-1\,safSg=2N\,safApp=OpenSAF,safSi=SC-2N,safApp=OpenSAF
        saAmfSISUHAState=ACTIVE(1)
  
```


# References
- [OpenSAF quick-start guide.](https://sourceforge.net/p/opensaf/wiki/OpenSAF%20quick-start%20guide%20%28simulated%20cluster%29)
- [Using OpenSAF as an application.](https://sourceforge.net/p/opensaf/wiki/OpenSAF%20as%20an%20application)
- [github.com/adrian77/docker](https://github.com/adrian77/docker)

If you prefer to install on virtual machine, you can use the below iso images:

- [Ubuntu 16.04](https://releases.ubuntu.com/16.04)
- [Debian 12.08](https://mirror.accum.se/debian-cd/12.8.0)


