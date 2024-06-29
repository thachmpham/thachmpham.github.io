---
title: "Docker"
bookToc: false
bookFlatSection: true
bookHidden: true
---


# Docker Cheatsheet

{{< columns >}}
## Hub
- Login
```sh
docker login -u <username>
```

- Search an image
```sh
docker search <image>
```

- Pull an image
```sh
docker pull <image> <path/url>
```

- Push an image
```sh
docker push <username>/<image>
```

<--->


## Images
- Build an image from a Dockerfile
```sh
docker build -t <image>
```

- List local images
```sh
docker images
```

- Delete an image
```sh
docker rmi <image>
```

{{< /columns >}}


## Containers
### General
- Create, run container from an image, with a custom name
```sh
docker run --name <container> <image>
```

- Open a terminal inside a running container
```sh
docker exec -it <container> sh
```

- Inspect a running container
```sh
docker inspect <container>
```

### Port
- Publish a container's port(s) to the host
```sh
docker run -p <host_port>:<container_port> <image>
```

- Publish all exposed ports to random ports
```sh
docker run -P <image>
```


### Volume
- Setup bind mount. Good for sharing data between host and container.
```sh
docker run -it --mount type=bind,source=<path>,target=<path> <image>
```

- Setup volume mount. Good for sharing data between containers.
```sh
docker run -it --mount type=volume,source=<path>,target=<path> <image>
```


## Dockerfile

{{< columns >}}
- `Dockerfile`
```Dockerfile
FROM ubuntu:latest
CMD echo 'hello'
```

<--->

- Run
```sh
docker build -t img_ubuntu .
docker run img_ubuntu
```

{{< /columns >}}

References:  
- [Dockerfile](https://docs.docker.com/reference/dockerfile)


## Compose
{{< columns >}}
- `docker-compose.yml`
```yml
name: myapp

services:
  foo:
    image: ubuntu
    command: echo hello
```

<--->

- Run
```sh
docker compose up

```

{{< /columns >}}

References:  
- [Compose](https://docs.docker.com/compose/compose-file)