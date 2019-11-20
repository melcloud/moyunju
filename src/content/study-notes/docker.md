---
title: "Docker"
date: 2018-06-08T16:06:45+10:00
draft: true
tags: []
---

## Docker Images

- A Docker image consists of read-only layers each of which represents a Dockerfile instruction. The layers are stacked and each one is a **delta of the changes from the previous layer**.
- `docker rmi` or `docker image rm` to remove image. Use `-a` to list all intermediate images. `docker rmi $(docker images -q)` to remove all images
- `docker tag <source>:<tag> <target>:<tag>` to tag a resource. The image ID will remain the same. Use `docker image rm <tag>` to remove a tag
- `docker build -t <repo_name>/<image_name>:<tag>` to tag a resource at build time
- `docker login -u user -p password <server>` to login to private repository
- `docker push` and `docker pull` to upload / download images. Only layers which are not cached will be pushed / pulled
- `docker image rm $(docker image ls -a -q)` remove all images

## Docker containers

- When you run an image and generate a container, you add a new writable layer (the “container layer”) on top of the underlying layers. All changes made to the running container, such as writing new files, modifying existing files, and deleting files, are written to this thin writable container layer.
- `docker ps` to list running containers. `docker ps -a` to list all containers. `docker rm $(docker ps -qa)` to remove all containers
- `docker run <options> <image> <command> <args>` is used to create a container
    - `-i --interactive` keeps stdin open even if not attached
    - `-d --detach` runs container in background and print container ID
    - `-t --tty` allocates a pseudo-tty
    - `-w --workdir` changes working directory
    - `--name` to assign a name
    - `-p --publish` to map port
    - `--publish-all` to all exposed ports to a random port on host
- `docker rename <old> <new>` to rename a container
- `docker stop` to stop a running container with default wait time 10 seconds. This can be changed by `-t`
- `docker start` to start a container
    - `-a` attaches STDIN and STDERR and forward signal. A ctrl + C will not terminate running container
    - `-i` attaches container's STDIN. A ctrl + C will terminate running container
- `docker restart` to restart a container with default wait time 10 seconds. This can be chaged by `-t`
- `docker exec <option> <container> <command>` to execute a command against running container
    - `-d` runs command in background
    - `-i` keeps container's STDIN open
    - `-t` allocates a pseudo-tty
    - `-u --user` changes user to the one specified by either the UID or username
    - `-e --env` supplies environment variables. The format is <name>="<value>"
- `docker container rm $(docker container ls -a -q) ` remove all containers

## Docker network

- `docker network create` to create new network
- `docker network connect <network name> <container>` to connect container to network
- `docker network inspect <network name>` to inspect network

## Docker compose

``` yml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      # Immediately restart containers if one fails
      restart_policy:
        condition: on-failure
    # Map port 4000 on the host to web’s port 80
    ports:
      - "4000:80"
    networks:
      # Instruct web’s containers to share port 80 via a load-balanced network called webnet. (Internally, the containers themselves publish to web’s port 80 at an ephemeral port.)
      - webnet
networks:
  # default load balanced network
  webnet:
```

## Docker machine
Use `docker-machine create --driver virtualbox myvm1` to create new virtual machine, and `docker-machine ssh myvm1` to ssh. `eval $(docker-machine env myvm1)` sets myvm1 to default active machine.

## Docker environment variables

- **DOCKER_HOST** gives the URL to connect to docker

## Security
Each container has its own control group, namespace and network stack. But they do share same Linux kernel.

## Best practice
### Use smaller image
Smaller images have following benefits:
- faster to pull down and start
- Less pre-installed applications improved security

In order to produce smaller images:
- Use multi-stage build or reduce RUN commands
- Use smaller base image

### Create your own base image
If you have multiple images with a lot in common, consider creating your own base image with the shared components. Common layers are only loaded once by docker

### Debugging
Keep production image lean and use it as a base image for debugging. Add debugging tools on top of it

### Tag image
The notation for associating a local image with a repository on a registry is username/repository:tag. Use `docker tag image username/repository:tag` to tag a image. Also tag images with codify version information, intended destination (prod or test, for instance), stability, or other information which is useful

### Storage
use *bind mounts* during development to mount your source directory or a binary. For production, use a *volume* instead, mounting it into the same location as bind mount.

### Multi-stage build pattern
