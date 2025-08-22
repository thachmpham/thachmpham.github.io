---
title:  'Docker Container with SSH Access'
---

Setup a docker container accessible via SSH from other hosts.

# 1. Docker Image
- Dockerfile.
```Dockerfile
  
# Use Ubuntu as base
FROM ubuntu:20.04

# Prevent interactive prompts
ENV DEBIAN_FRONTEND=noninteractive

# Install OpenSSH server
RUN apt-get update && \
    apt-get install -y openssh-server sudo && \
    mkdir /var/run/sshd

# Allow password authentication
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
    echo "PermitRootLogin yes" >> /etc/ssh/sshd_config

# Expose SSH port
EXPOSE 22

# Start SSH service
CMD ["/usr/sbin/sshd", "-D"]
  
```


# 2. Docker Container
- Run docker container with port forwarding, port 2222 on host binds to port 22 on container.
```sh
  
$ docker run -d -p 2222:22 -v /root/repo:/root/repo --name pc --hostname pc ubuntu20.04
  
```


# 3. SSH
```sh
  
$ ssh root@<host_ip> -p 2222
  
```