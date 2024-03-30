---
title: "TIPS"
bookToc: false
bookFlatSection: true
---
# TIPS

## 1. RPM
### 1.1. Rebuild rpm
**Rebuild**
```sh
$ wget https://kojipkgs.fedoraproject.org//packages/shntool/3.0.10/31.fc39/src/shntool-3.0.10-31.fc39.src.rpm
$ rpm -ivvh shntool-3.0.10-31.fc39.src.rpm
$ rpmbuild -ba ~/rpmbuild/SPECS/shntool.spec
$ rpm -Uvh --force ~/rpmbuild/RPMS/aarch64/shntool-3.0.10-31.fc39.aarch64.rpm
```

**Create & Apply Patch**
```sh
# extract source
$ cd ~/rpmbuild/SOURCES
$ tar -xvf shntool-3.0.10.tar.gz
$ cp -r shntool-3.0.10 shntool-3.0.10.orig
$ vi ./shntool-3.0.10/src/mode_info.c

# create patch
$ diff -Npru shntool-3.0.10.orig shntool-3.0.10 > hello.patch

# add patch to spec file
$ cd ~/rpmbuild/SPECS
$ vi shntool.spec
  # Patch5: hello.patch

# install
$ rpmbuild -ba ~/rpmbuild/SPECS/shntool.spec
$ rpm -Uvh --force ~/rpmbuild/RPMS/aarch64/shntool-3.0.10-31.fc39.aarch64.rpm
```
References:
- https://wiki.centos.org/HowTos(2f)RebuildSRPM.html
- https://src.fedoraproject.org/


## 2. Find
```sh
# find and execute
$ find ~/ -name '.*rc' -exec sh -c 'echo $1' _ {} \;


```