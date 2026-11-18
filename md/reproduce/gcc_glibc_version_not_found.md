---
title: 'GCC libc.so.6 Version GLIBC_X.XX Not Found'
subtitle: '(Reproduce & Troubleshoot Series)'
---


# Overview
Build a binary on a machine, run it on another machine. It is the classic recipe for the libc version mismatch issue.
In this blog, we will reproduce the issue by buiding a binary on fedora 44, and running it on fedora 33.

```sh
libc.so.6: version 'GLIBC_X.XX' not found
```


# Setup Lab
Find gcc and fedora version mapping at [koji.fedoraproject.org](https://koji.fedoraproject.org):

- [gcc-10.3.1-1.fc33.aarch64.rpm](https://koji.fedoraproject.org/koji/search?terms=%5Egcc-10.*&type=rpm&match=regexp): Fedora 33, GCC 10.
- [gcc-16.2.1-1.fc44.aarch64.rpm](https://koji.fedoraproject.org/koji/search?terms=%5Egcc-%5Cd%2B.*fc44&type=rpm&match=regexp): Fedora 44, GCC 16.


Step 1: Create docker containers of fedora 33 and fedora 44.
```sh
host$ git clone https://github.com/thachmpham/lab.git

host$ cd lab/c-development

host$ docker compose build
host$ docker compose up --detach
```

- To access fedora 33 container.
```sh
host$ docker exec -it f33 bash
```

- To access fedora 44 container.
```sh
host$ docker exec -it f44 bash
```


Step 2: Check glibc versions.

```sh
f33$ ldconfig -p | grep libc.so
libc.so.6 (libc6,AArch64, OS ABI: Linux 3.7.0) => /lib64/libc.so.6

f33$ /lib64/libc.so.6
GNU C Library (GNU libc) stable release version 2.32.
Compiled by GNU CC version 10.3.1 20210422 (Red Hat 10.3.1-1).
```

```sh
f44$ /lib64/libc.so.6
GNU C Library (GNU libc) stable release version 2.43.
Compiled by GNU CC version 16.2.1 20260819 (Red Hat 16.2.1-2).
Minimum supported kernel: 3.7.0
```


# Cannot Start Program Due to Missing Glibc Versions
## Reproduce
Step 1: Build a program on fedora 44.
```sh
f44$ vi main.c
```

```cpp
#include <stdio.h>

int main(int argc, char** argv)
{
	printf("hello\n");
	return 0;
}
```

```sh
f44$ g++ main.cpp -o main

f44$ ./main
hello
```

Step 2: Run the program on fedora 33.
```sh
f33$ ./main
./main: /lib64/libc.so.6: version `GLIBC_2.34' not found (required by ./main)

f33$ ldd main
	./main: /lib64/libc.so.6: version `GLIBC_2.34' not found (required by ./main)
    linux-vdso.so.1 (0x0000ffffaf8ae000)
    libc.so.6 => /lib64/libc.so.6 (0x0000ffffaf6da000)
    /lib/ld-linux-aarch64.so.1 (0x0000ffffaf860000)
```

The error means that libc.so.6 does not have GLIBC_2.34 which required by ./main.


## Analysis

Step 1: Check GLIBC versions that the program requires.
```sh
f33$ readelf --version-info main
Version needs section '.gnu.version_r' contains 1 entry:
 Addr: 0x0000000000400410  Offset: 0x000410  Link: 5 (.dynstr)
  000000: Version: 1  File: libc.so.6  Cnt: 2
  0x0010:   Name: GLIBC_2.17  Flags: none  Version: 3
  0x0020:   Name: GLIBC_2.34  Flags: none  Version: 2
```

Section '.gnu.version_r' contains the GLIBC versions that the program requires. The program needs GLIBC 2.17 and 2.34.

Step 3: Check GLIBC versions defined in libc.so.6.
```sh
f33$ readelf --version-info /lib64/libc.so.6
Version definition section '.gnu.version_d' contains 14 entries:
  000000: Rev: 1  Flags: BASE  Index: 1  Cnt: 1  Name: libc.so.6
  0x001c: Rev: 1  Flags: none  Index: 2  Cnt: 1  Name: GLIBC_2.17
  0x0038: Rev: 1  Flags: none  Index: 3  Cnt: 2  Name: GLIBC_2.18
  0x005c: Rev: 1  Flags: none  Index: 4  Cnt: 2  Name: GLIBC_2.22
  0x0080: Rev: 1  Flags: none  Index: 5  Cnt: 2  Name: GLIBC_2.23
  0x00a4: Rev: 1  Flags: none  Index: 6  Cnt: 2  Name: GLIBC_2.24
  0x00c8: Rev: 1  Flags: none  Index: 7  Cnt: 2  Name: GLIBC_2.25
  0x00ec: Rev: 1  Flags: none  Index: 8  Cnt: 2  Name: GLIBC_2.26
  0x0110: Rev: 1  Flags: none  Index: 9  Cnt: 2  Name: GLIBC_2.27
  0x0134: Rev: 1  Flags: none  Index: 10  Cnt: 2  Name: GLIBC_2.28
  0x0158: Rev: 1  Flags: none  Index: 11  Cnt: 2  Name: GLIBC_2.29
  0x017c: Rev: 1  Flags: none  Index: 12  Cnt: 2  Name: GLIBC_2.30
  0x01a0: Rev: 1  Flags: none  Index: 13  Cnt: 2  Name: GLIBC_2.32
  0x01c4: Rev: 1  Flags: none  Index: 14  Cnt: 2  Name: GLIBC_PRIVATE
```

Section '.gnu.version_d' contains the GLIBC versions defined by libc.so.6. It does not contain GLIBC 2.34 which required by the program.
So, the program cannot start on fedora 33 due to lack of the required version.


## Workaround: Modify Section ".gnu.version_r"
To let the program not require GLIBC 2.34 anymore, we will modify the .gnu.version_r section, replace GLIBC_2.34 by GLIBC_2.17.

Step 1: Patch the ELF.
```sh
f33$ vi patch.py
```

```python
import lief

elf = lief.parse('main')

# in section .gnu.version_r, change GLIBC_2.34 to GLIBC_2.17
elf.symbols_version_requirement[0].get_auxiliary_symbols()[1].name = "GLIBC_2.17"

# write elf to another file, main1.
elf.write('main1')
```

```sh
f33$ python3 patch.py

f33$ readelf --version-info main1
Version needs section '.gnu.version_r' contains 1 entry:
 Addr: 0x0000000000400410  Offset: 0x000410  Link: 5 (.dynstr)
  000000: Version: 1  File: libc.so.6  Cnt: 2
  0x0010:   Name: GLIBC_2.17  Flags: none  Version: 3
  0x0020:   Name: GLIBC_2.17  Flags: none  Version: 2
```

Step 2: Check dependencies and run the program.
```sh
f33$ ldd main1
	linux-vdso.so.1 (0x0000ffffa2884000)
	libc.so.6 => /lib64/libc.so.6 (0x0000ffffa26ba000)
	/lib/ld-linux-aarch64.so.1 (0x0000ffffa2840000)

f33$ chmod +x main1

f33$ ./main1
hello
```


# References
- [Multiple glibc on a Single Host](https://stackoverflow.com/questions/847179/multiple-glibc-libraries-on-a-single-host).
