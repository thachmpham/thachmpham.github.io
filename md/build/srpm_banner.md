---
title: 'Fedora SRPM: Banner'
subtitle: '(Build Series)'
---


# Overview


# Prepare Lab
- Clone code.
```sh
host$ git clone https://github.com/thachmpham/lab.git
```


- Build container.
```sh
host$ cd build/rpm
host$ docker compose build
host$ docker compose up --detach
```


- Access container.
```sh
host$ docker exec -it rpm bash
```


# Banner
## Clone
- Clone project.
```sh
$ git clone https://src.fedoraproject.org/rpms/banner.git
$ cd banner
$ git switch f44

$ tree
.
├── banner.spec
└── sources
```


- Download source code.
```sh
$ spectool -g banner.spec
Downloaded: banner-1.3.6.tar.gz
```


## Prepare
- Prepare rpmbuild directory.
```sh
$ cp banner-1.3.6.tar.gz ~/rpmbuild/SOURCES/

# extract ~/rpmbuild/SOURCES/ to ~/rpmbuild/BUILD
$ rpmbuild -bp banner.spec

$ tree ~/rpmbuild/
/root/rpmbuild/
├── BUILD
│   └── banner-1.3.6-build
│       └── banner-1.3.6
│           ├── Makefile.in
│           ├── banner.c
│           └── configure
├── RPMS
├── SOURCES
    └── banner-1.3.6.tar.gz
```


## Build
- Build rpm.
```sh
$ rpmbuild -bb --noprep --noclean banner.spec

$ tree ~/rpmbuild/
/root/rpmbuild/
├── BUILD
│   └── banner-1.3.6-build
│       └── banner-1.3.6
│           ├── Makefile.in
│           ├── banner.c
│           └── configure
├── RPMS
│   └── aarch64
│       ├── banner-1.3.6-1.fc44.aarch64.rpm
│       ├── banner-debuginfo-1.3.6-1.fc44.aarch64.rpm
│       └── banner-debugsource-1.3.6-1.fc44.aarch64.rpm
├── SOURCES
    └── banner-1.3.6.tar.gz
```


## Install
- Install rpm.
```sh
$ rpm -iv ~/rpmbuild/RPMS/aarch64/banner-1.3.6-1.fc44.aarch64.rpm
banner-1.3.6-1.fc44.aarch64

$ rpm -ql banner-1.3.6-1.fc44.aarch64
/usr/bin/banner
/usr/share/doc/banner
/usr/share/man/man1/banner.1.gz
```


## Run
- Run banner.
```sh
$ /usr/bin/banner 'hello'

#     #  #######  #        #        #######
#     #  #        #        #        #     #
#     #  #        #        #        #     #
#######  #####    #        #        #     #
#     #  #        #        #        #     #
#     #  #        #        #        #     #
#     #  #######  #######  #######  #######

$ man banner
NAME
       banner - prints a short string to the console in very large letters
```


# References
- [Fedora Package Sources](https://src.fedoraproject.org)
- [SRPM Banner](https://docs.fedoraproject.org/en-US/package-maintainers/Packaging_Tutorial_1_banner)
- [SRPM Hello](https://docs.fedoraproject.org/en-US/package-maintainers/Packaging_Tutorial_2_GNU_Hello)