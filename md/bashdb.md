---
title: BashDB - Build & Install
---

# 1. Introduction
Bashdb is a debugger for Bash scripts, allowing interactive debugging with features like breakpoints, step execution, variable inspection, and stack tracing. It's ideal for troubleshooting and analyzing complex scripts.


# 2. Build
## 2.1. Docker Container
We will create a docker container that will be used to build bashdb.

- Create Dockerfile.
```Dockerfile
  
FROM fedora:24

RUN dnf -y install rpm-build rpm-devel rpmlint make rpmdevtools
RUN dnf -y install autoconf automake texinfo
RUN dnf -y install wget
  
```

- Build docker image.
```sh
  
$ docker build --progress=plain -t fedora.24 .
  
```

- Create docker container.
```sh
  
$ docker run -it -v /root:/root fedora.24 bash
  
```

## 2.2. Build
We will download bashdb source code and build in the docker container.

- Create rpmbuild directory.
```sh
  
$ rpmdev-setuptree
  
```

- Download source code.
```sh
  
$ cd /root/rpmbuild/SOURCES

$ wget --no-check-certificate https://sourceforge.net/projects/bashdb/files/bashdb/4.3-0.91/bashdb-4.3-0.91.tar.gz
  
```

- Create rpm spec file `/root/rpmbuild/SPECS/bashdb.spec`.
```sh
  
Name:           bashdb
Version:        4.3
Release:        0
Summary:        bashdb debugger
License:        GPLv2+

BuildArch:      noarch
Source0:        bashdb-4.3-0.91.tar.gz


%description
debugger for bash script


%prep
tar xf /root/rpmbuild/SOURCES/bashdb-4.3-0.91.tar.gz


%build
mkdir -p /opt/bashdb
cd bashdb-4.3-0.91
./configure --prefix=/opt/bashdb
make


%install
cd bashdb-4.3-0.91
make install
mkdir -p %{buildroot}/opt/bashdb
cp -r /opt/bashdb/bin %{buildroot}/opt/bashdb
cp -r /opt/bashdb/share %{buildroot}/opt/bashdb


%files
/opt/bashdb
  
```

- Build rpm package.
```sh
  
$ rpmbuild -bb /root/rpmbuild/SPECS/bashdb.spec
  
```

## 2.3. Install
- Copy rpm to shared folder.
```sh
  
$ cp /root/rpmbuild/RPMS/noarch/bashdb-4.3-0.noarch.rpm /root
  
```

- Install rpm.
```sh
  
$ rpm -ivh --nodeps bashdb-4.3-0.noarch.rpm
  
```

- Run bashdb.
```sh
  
$ /opt/bashdb/bin/bashdb --help
  
```


# 3. Manual
## 3.1. Start
Start a script under bashdb control.
```sh
  
$ bashdb /usr/bin/ldd
  
```

## 3.2. Redirect Output
Redirect output to another terminal.

- Get path of terminal 1.
```sh
  
term1$ tty
/dev/pts/1
  
```

- From terminal 0, redirect output to terminal 1.
```sh
  
term0$ bashdb /usr/bin/ldd > /dev/pts/1
  
```


## 3.3. Set Breakpoint In Code
Hardcode a breakpoint at a line of code.

- Add the below code to the bash script.
```sh
  
source /opt/bashdb/share/bashdb/bashdb-trace -L /opt/bashdb/share/bashdb
_Dbg_debugger
  
```


# 4. Issues
### 4.1. Could not found /usr/bin/bash
Problem.
```sh
  
bashdb: /usr/bin/bash: bad interpreter: No such file or directory
  
```

Solution.
```sh
  
$ ln -s /bin/bash /usr/bin/bash
  
```

### 4.2. Argument $0 is bashdb, but the expected $0 is the target script.
Solution: Invoke bashdb from the script.
```sh
  
# add the below line to the script
source /opt/bashdb/share/bashdb/bashdb-trace -L /opt/bashdb/share/bashdb
_Dbg_debugger
  
```

# References
- [bashdb.sourceforge.net](https://bashdb.sourceforge.net/)
- [bashdb.readthedocs.io](https://bashdb.readthedocs.io)
