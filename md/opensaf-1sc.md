---
title:  'Setup Cluster - 1 SC'
---


# 1. Introduction.
- Build a docker image with OpenSAF pre-installed.
- Launch a container using this docker image.
- Configure the container to act as an OpenSAF system controller.


# 2. Build Docker Image.
Create a `Dockerfile` with the following content.
```Dockerfile
  
FROM ubuntu:16.04

RUN apt update
RUN apt install -y \
    sudo sqlite3 libxml2 psmisc \
    python2.7-minimal net-tools kmod
RUN apt install -y \
    mercurial gcc g++ libxml2-dev automake \
    m4 autoconf libtool pkg-config \
    make python-dev libsqlite3-dev binutils
RUN apt install -y \
    vim iputils-ping rsyslog

RUN cd /root && hg clone http://hg.code.sf.net/p/opensaf/staging opensaf-staging

WORKDIR /root/opensaf-staging
RUN ./bootstrap.sh
RUN ./configure --enable-tipc CPPFLAGS="-DRUNASROOT"
RUN make -j `nproc`
RUN make install
RUN ldconfig
  
```

Build docker image.
```sh
  
$ docker build --progress=plain -t opensaf .
  
```

# 3. Launch Docker Container.
```sh
  
$ docker run --privileged -it --name node1 -h SC-1 opensaf
  
```

# 4. Configure.
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
- https://github.com/adrian77/docker

