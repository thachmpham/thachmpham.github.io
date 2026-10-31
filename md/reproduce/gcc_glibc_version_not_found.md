---
title: 'GCC libc.so.6 Version GLIBC_X.XX Not Found'
subtitle: '(Reproduce & Troubleshoot Series)'
---


# Overview


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
RUN dnf group install -y c-development
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
RUN dnf group install -y c-development
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