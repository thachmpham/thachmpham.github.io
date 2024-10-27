---
title:  'Setup Cluster - 2 SCs'
---


# 1. Introduction.
- Build a docker image with OpenSAF pre-installed.
- Launch 2 container using this docker image.
- Configure the 2 containers to act as system controllers within the same cluster.


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

# 3. Setup SC-1.
## 3.1. Launch Docker Container.
```sh
  
$ docker run --privileged -it --name node1 -h SC-1 opensaf
  
```


## 3.2. Configure.
Generate `/etc/opensaf/imm.xml`.
```sh
  
$ cd /usr/local/share/opensaf/immxml/

# generate nodes.cfg
$ ./immxml-clustersize --sc-count 2
Successful, result stored in the file ./nodes.cfg

$ ./immxml-configure
Successfully generated the imm file: ./imm.xml.20241015_1721

$ cp ./imm.xml.20241015_1721 /etc/opensaf/imm.xml
  
```

Edit `/etc/opensaf/node_name`.
```sh
  
SC-1
# SC-1: hostname.
  
```

Edit `/etc/opensaf/nid.conf`.
```sh
  
# MDS transport protocol, values supported are TIPC or TCP
export MDS_TRANSPORT=TCP
  
```

Edit `/etc/opensaf/dtmd.conf`.
```sh
  
DTM_NODE_IP=172.17.0.2
# 172.17.0.2: IP of the eth0 interface.
  
```

Edit `/etc/opensaf/slot_id`.
```sh
  
1
  
```

Edit `/etc/opensaf/node_type`.
```sh
  
controller
  
```


## 3.3. Start OpenSAF.
Start OpenSAF.
```sh
   
$ /etc/init.d/opensafd start
Starting OpenSAF Services (Using TCP): *
  
```

Check status.
```sh
  
$ /etc/init.d/opensafd status
safSISU=safSu=SC-1\,safSg=NoRed\,safApp=OpenSAF,safSi=NoRed1,safApp=OpenSAF
        saAmfSISUHAState=ACTIVE(1)
safSISU=safSu=SC-1\,safSg=2N\,safApp=OpenSAF,safSi=SC-2N,safApp=OpenSAF
        saAmfSISUHAState=ACTIVE(1)
  
```


# 4. Setup SC-2.
## 4.1. Launch Docker Container.
```sh
  
$ docker run --privileged -it --name node2 -h SC-2 opensaf
  
```


## 4.2. Configure.
Edit `/etc/opensaf/node_name`.
```sh
  
SC-2
# SC-2: hostname.
  
```

Edit `/etc/opensaf/nid.conf`.
```sh
  
# MDS transport protocol, values supported are TIPC or TCP
export MDS_TRANSPORT=TCP
  
```

Edit `/etc/opensaf/dtmd.conf`.
```sh
  
DTM_NODE_IP=172.17.0.3
# 172.17.0.3: IP of the eth0 interface.
  
```

Edit `/etc/opensaf/slot_id`.
```sh
  
2
  
```

Edit `/etc/opensaf/node_type`.
```sh
  
controller
  
```


## 4.3. Start OpenSAF.
Start rsyslog because opensaf print logs to rsyslog.
```sh
  
$ /etc/init.d/rsyslog start
  
```

Start opensaf.
```sh
   
$ /etc/init.d/opensafd start
Starting OpenSAF Services (Using TCP): *
  
```

Check status.
```sh
  
$ /etc/init.d/opensafd status
safSISU=safSu=SC-1\,safSg=2N\,safApp=OpenSAF,safSi=SC-2N,safApp=OpenSAF
        saAmfSISUHAState=ACTIVE(1)
safSISU=safSu=SC-1\,safSg=NoRed\,safApp=OpenSAF,safSi=NoRed1,safApp=OpenSAF
        saAmfSISUHAState=ACTIVE(1)
safSISU=safSu=SC-2\,safSg=NoRed\,safApp=OpenSAF,safSi=NoRed2,safApp=OpenSAF
        saAmfSISUHAState=ACTIVE(1)
safSISU=safSu=SC-2\,safSg=2N\,safApp=OpenSAF,safSi=SC-2N,safApp=OpenSAF
        saAmfSISUHAState=STANDBY(2)
  
```


# 5. Test.
Check state of nodes.
```sh
  
$ amf-state node
safAmfNode=SC-1,safAmfCluster=myAmfCluster
        saAmfNodeAdminState=UNLOCKED(1)
        saAmfNodeOperState=ENABLED(1)
safAmfNode=SC-2,safAmfCluster=myAmfCluster
        saAmfNodeAdminState=UNLOCKED(1)
        saAmfNodeOperState=ENABLED(1)
  
```

Check `/var/log/syslog` on SC-1.
```sh
  
SC-1 osafdtmd[133]: NO Established contact with 'SC-2'
SC-1 osafamfd[224]: NO Node 'SC-2' joined the cluster
  
```

Check `/var/log/syslog` on SC-2.
```sh
  
SC-2 osafdtmd[70]: NO Established contact with 'SC-1'
SC-2 osafclmna[83]: NO safNode=SC-2,safCluster=myClmCluster Joined cluster, nodeid=2020f
SC-2 osafamfd[154]: NO Cold sync complete!
  
```


# References
- [OpenSAF quick-start guide.](https://sourceforge.net/p/opensaf/wiki/OpenSAF%20quick-start%20guide%20%28simulated%20cluster%29)
- [Using OpenSAF as an application.](https://sourceforge.net/p/opensaf/wiki/OpenSAF%20as%20an%20application)
- https://github.com/adrian77/docker

