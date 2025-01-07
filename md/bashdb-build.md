
```Dockerfile
  
FROM fedora:24

RUN dnf -y install rpm-build rpm-devel rpmlint make rpmdevtools
RUN dnf -y install autoconf automake texinfo
RUN dnf -y install wget git
  
```


```sh
  
docker build --progress=plain -t fedora.24 .
  
```


```sh
  
rpmdev-setuptree

cd /root/rpmbuild/SOURCES

wget --no-check-certificate https://sourceforge.net/projects/bashdb/files/bashdb/4.3-0.91/bashdb-4.3-0.91.tar.gz
  
```


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


```sh
  
rpmbuild -bp /root/rpmbuild/SPECS/bashdb.spec
  
```