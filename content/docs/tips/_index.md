---
title: "TIPS"
bookToc: false
bookFlatSection: true
---
# TIPS

## 1. RPMS
### 1.1. Rebuild rpm
```sh
$ wget https://kojipkgs.fedoraproject.org//packages/shntool/3.0.10/31.fc39/src/shntool-3.0.10-31.fc39.src.rpm
$ rpm -ivvh shntool-3.0.10-31.fc39.src.rpm
$ rpmbuild -ba ~/rpmbuild/SPECS/shntool.spec
$ find ~/rpmbuild/RPMS/aarch64 -name '*.rpm' -exec sh -c 'rpm -Uvh --force $1' _ {} \;
```
- https://wiki.centos.org/HowTos(2f)RebuildSRPM.html
- https://src.fedoraproject.org/

