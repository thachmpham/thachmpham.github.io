---
title: "TIPS"
bookToc: true
bookFlatSection: true
bookHidden: true
---
# TIPS

## 1. RPM
**1.1. Rebuild rpm**
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
# + Patch5: hello.patch

# install
$ rpmbuild -ba ~/rpmbuild/SPECS/shntool.spec
$ rpm -Uvh --force ~/rpmbuild/RPMS/aarch64/shntool-3.0.10-31.fc39.aarch64.rpm
```

References:
- https://wiki.centos.org/HowTos(2f)RebuildSRPM.html
- https://src.fedoraproject.org/


## 2. Find
- Find, execute.
```sh
$ find ~/ -name '.*rc' -exec sh -c 'echo $1' _ {} \;
```

## 3. C++
- Println.
```c++
#define print(fmt, args...) printf("%s: " fmt "\n", __func__, ## args)
```

## 4. IO
- TCP listening sockets.
```sh
$ ss -ntl
```
- Port status
```sh
$ nmap -p 389 172.16.111.130
```

## 5. Firewall
- Allow incoming traffic
```sh
$ firewall-cmd --permanent --add-port=389/tcp
$ firewall-cmd --reload
```

## 6. KVM
- Create Ubuntu VM
```sh
$ virt-install \
--name vm-ubuntu --os-variant ubuntu20.04 \
--location http://ftp.ubuntu.com/ubuntu/dists/focal/main/installer-amd64 \
--vcpus 8 --ram 8192 \
--network bridge=virbr0,model=virtio \
--graphics none \
--extra-args='console=ttyS0,115200n8 serial'
```

- Fix: virsh console hangs at promt "The escape character ^]"
```sh
$ virsh net-list
Name    State   Autostart   Persistent
default active  yes         yes

$ virsh net-dhcp-leases default
IP address              Hostname
192.168.123.35/24       ubuntu
```

```sh
$ ssh root@192.168.123.35

$ vi /etc/default/grub
GRUB_TERMINAL=serial
GRUB_SERIAL_COMMAND="serial --unit=0 --speed=115200 --word=8 --parity=no --stop=1"

$ update-grub
$ reboot
```

```sh
$ virsh console vm-ubuntu
```

- Create Fedora VM
```
virt-install \
--name vm-fedora \
--location https://repo.jing.rocks/fedora-buffet/fedora/linux/releases/39/Server/x86_64/os \
--network bridge=virbr0,model=virtio \
--graphics none \
--extra-args='console=ttyS0,115200n8 serial'
```

Distro urls:  
- Fedora/RedHat: http://download.fedoraproject.org/pub/fedora/linux/releases/10/Fedora/i386/os/
- Debian/Ubuntu: http://ftp.us.debian.org/debian/dists/etch/main/installer-amd64/
- Suse: http://download.opensuse.org/distribution/11.0/repo/oss/
- Mandriva: ftp://ftp.uwsg.indiana.edu/linux/mandrake/official/2009.0/i586/


## 7. Terminal
- Attach a running process to a new terminal.
```sh
reptyr PID
```
- https://blog.nelhage.com/2011/01/reptyr-attach-a-running-process-to-a-new-terminal/

## 8. Jupyter Remotely
- Setup Server Which Runs Jupyter
```sh
$ pip install jupyterlab
```

```sh
$ jupyter lab --generate-config
Writing default config to: /root/.jupyter/jupyter_lab_config.py
```

- /root/.jupyter/jupyter_lab_config.py
```py
c.ServerApp.allow_root = True

c.ServerApp.open_browser = False
c.ExtensionApp.open_browser = False

c.ServerApp.ip = '0.0.0.0'
c.ServerApp.port = 8888
```

- Setup Client (Localhost)
```sh
ssh -L localhost:8888:172.16.111.130:8888 root@172.16.111.130
```


