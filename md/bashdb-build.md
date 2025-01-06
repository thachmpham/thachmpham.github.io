
```Dockerfile
FROM bash:4.3 AS bash-stage

FROM fedora

COPY --from=bash-stage /usr/local/bin/bash /bin/
COPY --from=bash-stage /lib/ld-musl-aarch64.so.1 /lib
COPY --from=bash-stage /lib/ld-musl-aarch64.so.1 /lib
COPY --from=bash-stage /usr/lib/libncursesw.so.6 /usr/lib

RUN dnf -y update
RUN dnf -y install gcc rpm-build rpm-devel rpmlint make python coreutils rpmdevtools
RUN dnf -y install autoconf automake texinfo

CMD ["/bin/bash"]
```


```sh
docker build --progress=plain -t bashdb-builder .
```