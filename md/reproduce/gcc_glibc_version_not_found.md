---
title: 'GCC libc.so.6 Version GLIBC_X.XX Not Found'
subtitle: '(Reproduce & Troubleshoot Series)'
---


# Overview
Build a binary on a machine, run it on another machine. It is the classic recipe for the libc version mismatch issue. Let's reproduce and troubleshoot the issue.

```sh
libc.so.6: version 'GLIBC_X.XX' not found
```

# Reproduce
Find gcc and fedora version mapping at [koji.fedoraproject.org](https://koji.fedoraproject.org):

- [gcc-10.3.1-1.fc33.aarch64.rpm](https://koji.fedoraproject.org/koji/search?terms=%5Egcc-10.*&type=rpm&match=regexp).
- [gcc-16.2.1-1.fc44.aarch64.rpm](https://koji.fedoraproject.org/koji/search?terms=%5Egcc-%5Cd%2B.*fc44&type=rpm&match=regexp).

Through the RPM names, we have the below mapping:

- GCC 10, Fedora 33.
- GCC 16, Fedora 44.


Step 1: Create docker container of fedora 33.
```sh
host$ cat Dockerfile.f33
```
```Dockerfile
FROM fedora:33
RUN dnf group install -y c-development patchelf
CMD ["tail", "-f", "/dev/null"]
```

```sh
host$ docker build --tag fedora33 --file Dockerfile.f33 .
host$ docker create --name f33 --hostname f33 fedora33
host$ docker start f33
```


Step 2: Create docker container of fedora 44.
```sh
host$ cat Dockerfile.f44
```

```Dockerfile
FROM fedora:33
RUN dnf group install -y c-development patchelf
CMD ["tail", "-f", "/dev/null"]
```

```sh
host$ docker build --tag fedora44 --file Dockerfile.f44 .
host$ docker create --name f44 --hostname f44 fedora44
host$ docker start f44
```

Step 3: Check gcc and libc version on the containers.

- Fedora 33, g++ 10, libc 2.32.
```sh
f33$ g++ --version
g++ (GCC) 10.3.1 20210422 (Red Hat 10.3.1-1)

f33$ ldd --version
ldd (GNU libc) 2.32
```

- Fedora 44, g++ 16, libc 2.43.
```sh
f44$ g++ --version
g++ (GCC) 16.2.1 20260819 (Red Hat 16.2.1-2)

f44$ ldd --version
ldd (GNU libc) 2.43
```

Step 4: On fedora 44, build a binary.

- Build.
```sh
f44$ cat main.cpp
```

```cpp
#include <iostream>

int main()
{
    std::cout << "hello" << std::endl;
    return 0;
}
```

```sh
f44$ g++ main.cpp -o main

f44$ ./main
hello
```

Step 5: On fedora 33, run the binary which is built from fedora 44.

- Copy the binary to fedora 33.
```sh
host$ docker cp f44:/root/main .
host$ docker cp main f33:/root
```

- Fedora 33 cannot run the binary.
    - The binary requires GLIBC_2.34, which is not available in libc.so.6 on fedora 33.
    - The binary requires GLIBCXX_3.4.32, which is not available in libstdc++.so.6 on fedora 33.

```sh
f33$ ./main
./main: /lib64/libc.so.6: version `GLIBC_2.34' not found (required by ./main)
./main: /lib64/libstdc++.so.6: version `GLIBCXX_3.4.32' not found (required by ./main)
```


# Root Cause Analysis
- The binary depends on libc.so.6 and libstdc++.so.6.
```sh
f33$ ldd main
./main: /lib64/libc.so.6: version `GLIBC_2.34' not found (required by ./main)
./main: /lib64/libstdc++.so.6: version `GLIBCXX_3.4.32' not found (required by ./main)
	linux-vdso.so.1 (0x0000ffff8467c000)
	libstdc++.so.6 => /lib64/libstdc++.so.6 (0x0000ffff8442a000)
	libm.so.6 => /lib64/libm.so.6 (0x0000ffff84379000)
	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x0000ffff84348000)
	libc.so.6 => /lib64/libc.so.6 (0x0000ffff841c2000)
	/lib/ld-linux-aarch64.so.1 (0x0000ffff84630000)
```

- The binary requires:
    - GLIBC_2.17, GLIBC_2.34.
    - GLIBCXX_3.4, GLIBCXX_3.4.32.

```sh
f44$ strings main | grep GLIBC
GLIBC_2.17
GLIBC_2.34
GLIBCXX_3.4.32
GLIBCXX_3.4
__libc_start_main@GLIBC_2.34
_ZSt4cout@GLIBCXX_3.4
_ZSt21ios_base_library_initv@GLIBCXX_3.4.32
abort@GLIBC_2.17
...
```

- List of GLIBC supported on fedora 33. GLIBC_2.34 is not in the list.
```sh
f33$ strings /lib64/libc.so.6 | grep GLIBC
GLIBC_2.17
GLIBC_2.18
...
GLIBC_2.30
GLIBC_2.32
```

- List of GLIBCXX supported on fedora 33. GLIBCXX_3.4.32 is not in the list.
```sh
f33$ strings /lib64/libstdc++.so.6 | grep GLIBCXX
GLIBCXX_3.4
GLIBCXX_3.4.1
...
GLIBCXX_3.4.27
GLIBCXX_3.4.28
```

**Root Cause**

- On fedora 44, the binary is built with libc 2.43. It requires GLIBC_2.34.
- On fedora 33, libc 2.32 does not provide GLIBC_2.34. So, fedora 33 cannot run the binary due to lack of dependencies.


# Solution 1: Recompile on Runtime Machine
If that is the case you can setup a build environment on fedora 33, you can recompile the code on fedora 33 and run.If you can set up a build environment on Fedora 33, you can recompile the code directly on Fedora 33 and run the resulting binary in that environment.


# Solution 2: Recompile on Build Machine with rpath
If you cannot set up a build environment on Fedora 33 and must build the code on Fedora 44, you can use rpath option to specify where the runtime loader should look for dynamic libraries at runtime. This allows you to bypass the standard system library search paths (e.g., /lib64).

Step 1: On fedora 44, recompile with rpath.
```sh
f44$ g++ main.cpp -o main -Wl,--rpath=/root/patchlib -Wl,--dynamic-linker=/root/patchlib/ld-linux-aarch64.so.1

f44$ readelf --dynamic main
  Tag        Type                         Name/Value
 0x000000000000001d (RUNPATH)            Library runpath: [/root/patchlib]

f44$ readelf --segments main
  INTERP         0x0000000000000294 0x0000000000400294 0x0000000000400294
                 0x0000000000000025 0x0000000000000025  R      0x1
  [Requesting program interpreter: /root/patchlib/ld-linux-aarch64.so.1]
```

Step 2: Copy libc and interpreter of fedora 44 to fedora 33.
```sh
f33$ mkdir /root/patchlib

host$ docker cp f44:/lib64/libc.so.6 .
host$ docker cp f44:/lib64/libstdc++.so.6 .
host$ docker cp f44:/lib64/libstdc++.so.6.0.36 .
host$ docker cp f44:/lib/ld-linux-aarch64.so.1 .

host$ docker cp libc.so.6 f33:/root/patchlib
host$ docker cp libstdc++.so.6 f33:/root/patchlib
host$ docker cp libstdc++.so.6.0.36 f33:/root/patchlib
host$ docker cp ld-linux-aarch64.so.1 f33:/root/patchlib
```


Step 3: Copy the binary to fedora 33 and run.
```sh
host$ docker cp f44:/root/main .
host$ docker cp main f33:/root
```

```sh
f33$ ldd main
./main: /lib/ld-linux-aarch64.so.1: version `GLIBC_2.35' not found (required by /root/patchlib/libc.so.6)
	linux-vdso.so.1 (0x0000ffff9f594000)
	libstdc++.so.6 => /root/patchlib/libstdc++.so.6 (0x0000ffff9f2ab000)
	libm.so.6 => /lib64/libm.so.6 (0x0000ffff9f1fa000)
	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x0000ffff9f1c9000)
	libc.so.6 => /root/patchlib/libc.so.6 (0x0000ffff9f000000)
	/root/patchlib/ld-linux-aarch64.so.1 => /lib/ld-linux-aarch64.so.1 (0x0000ffff9f550000)

f33$ ./main
hello
```


# Workaround: Use patchelf to Edit rpath of Binary
If you cannot rebuild the code and only have the precompiled binary, you can use patchelf to modify the rpath of the binary. This workaround has the same effect as solution 2.

```sh
f44$ patchelf --set-interpreter /root/patchlib/ld-linux-aarch64.so.1 --force-rpath --set-rpath /root/patchlib main

f44$ patchelf --print-interpreter main
/root/patchlib/ld-linux-aarch64.so.1

f44$ patchelf --print-rpath main
/root/patchlib
```


# References
- [Multiple glibc on a Single Host](https://stackoverflow.com/questions/847179/multiple-glibc-libraries-on-a-single-host).
