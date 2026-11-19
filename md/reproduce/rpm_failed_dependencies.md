---
title: 'RPM: Failed Dependencies Error'
subtitle: '(Open Source Rebuild Series)'
---


# Overview
The rpm failed dependencies error occurs when we install a rpm which requires another rpm which is not installed in the system.
```sh
$ rpm --install /root/rpmbuild/RPMS/aarch64/banner-1.3.6-1.fc44.aarch64.rpm
error: Failed dependencies:
        tree is needed by banner-1.3.6-1.fc44.aarch64
```


# Setup Lab
- Setup lab [Fedora SRPM: Banner](/html/build/srpm_banner.html).


# Reproduce

:::::::::::::: {.columns}
::: {.column width=50%}

Step 1: Edit spec file to let banner rpm depend on tree rpm.
```sh
cont$ cd ~/banner
cont$ tree
.
├── banner-1.3.6.tar.gz
└── banner.spec

cont$ vi banner.spec
# -------------------- #
...
Requires:       tree
...
# -------------------- #
```

Step 2: Build banner rpm.
```sh
cont$ rpmbuild -bp banner.spec

cont$ rpmbuild -bb --noprep --noclean banner.spec
Wrote: /root/rpmbuild/RPMS/aarch64/banner-1.3.6-1.fc44.aarch64.rpm
```

:::
::: {.column width=50%}

Step 3: Check dependencies of banner rpm.
```sh
cont$ rpm -qR /root/rpmbuild/RPMS/aarch64/banner-1.3.6-1.fc44.aarch64.rpm
ld-linux-aarch64.so.1()(64bit)
ld-linux-aarch64.so.1(GLIBC_2.17)(64bit)
libc.so.6()(64bit)
...
rtld(GNU_HASH)
tree
```

Step 4: Remove tree rpm which required by banner rpm.
```sh
cont$ rpm -qa | grep tree
tree-2.2.1-4.fc44.aarch64

cont$ rpm --erase tree-2.2.1-4.fc44.aarch64
```

Step 5: Install banner rpm.
```sh
cont$ rpm --install /root/rpmbuild/RPMS/aarch64/banner-1.3.6-1.fc44.aarch64.rpm
error: Failed dependencies:
        tree is needed by banner-1.3.6-1.fc44.aarch64
```

:::
::::::::::::::


# Trick: RpmRebuild
In case we don't have the source to rebuild the banner rpm, we can modify the
final rpm using rpmrebuild, remove the failed dependencies.



:::::::::::::: {.columns}
::: {.column width=50%}

Step 1: Install rpmbuild.
```sh
cont$ dnf install rpmrebuild
```

Step 2: Remove the failed dependencies.
```sh
cont$ rpmrebuild -enp /root/rpmbuild/RPMS/aarch64/banner-1.3.6-1.fc44.aarch64.rpm
# -------------------- #
Requires:      ld-linux-aarch64.so.1()(64bit)
Requires:      ld-linux-aarch64.so.1(GLIBC_2.17)(64bit)
Requires:      libc.so.6()(64bit)
Requires:      rtld(GNU_HASH)
#Requires:      tree
# -------------------- #
```

:::
::: {.column width=50%}

Step 3: Check dependencies.
```sh
cont$ rpm -qR /root/rpmbuild/RPMS/aarch64/banner-1.3.6-1.fc44.aarch64.rpm
ld-linux-aarch64.so.1()(64bit)
ld-linux-aarch64.so.1(GLIBC_2.17)(64bit)
libc.so.6()(64bit)
...
rtld(GNU_HASH)
```

Step 4: Install the banner rpm.
```sh
cont$ rpm --install /root/rpmbuild/RPMS/aarch64/banner-1.3.6-1.fc44.aarch64.rpm
```

The banner rpm does not require tree rpm anymore, so we can proceed with the installation.

:::
::::::::::::::


# Solution: Install Needed Packages
Install tree rpm which is required by banner rpm.
```sh
cont$ dnf install tree
```


# References
- [RPM Spec Format](https://rpm.org/docs/4.20.x/manual/spec.html).
- [Modify RPM with RpmRebuild](https://linuxconfig.org/how-to-modify-an-rpm-package-using-rpmrebuild).
