---
title: BASHDB - Build & Install
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
RUN dnf -y install wget git
  
```

- Build docker image.
```sh
  
$ docker build --progress=plain -t fedora.24 .
  
```

- Create docker container.
```sh
  
$ docker run -it fedora.24 bash
  
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
mkdir -p /build
cd bashdb-4.3-0.91
./configure --prefix=/build
make


%install
cd bashdb-4.3-0.91
make install
mkdir -p %{buildroot}/usr
mkdir -p %{buildroot}/usr/share
cp -r /build/bin %{buildroot}/usr
cp -r /build/share/bashdb %{buildroot}/usr/share


%files
/usr/bin/bashdb
/usr/share/bashdb
  
```

- Build rpm package.
```sh
  
$ rpmbuild -bb /root/rpmbuild/SPECS/bashdb.spec
  
```


# References
- [bashdb.sourceforge.net](https://bashdb.sourceforge.net/)
- [bashdb.readthedocs.io](https://bashdb.readthedocs.io)