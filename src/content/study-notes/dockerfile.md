---
title: "Dockerfile"
date: 2018-06-08T16:07:40+10:00
draft: true
tags: ["study note"]
---

# Dockerfile structure

``` dockerfile
# This is a comment
FROM ubuntu:18.10

LABEL maintainer "Me"

RUN yum install apache
```

The `LABEL` adds metadata to the layer. The `FROM` and `RUN` will create temporary containers which normally referred as layers. Layers will be cached. If the instruction is not changed, then cached version will be used.

## COPY v.s. ADD

`COPY` is used to copy local directories / files to the given destination. It can be either relative or absolute. If it is relative, then it is relative to the working directory defined by `WORKDIR`.

``` dockerfile
# This is a comment
FROM ubuntu:18.10

LABEL maintainer "Me"

RUN yum install apache

COPY my-files/*.html /var/wwww
COPY ["my files/*.html", "/var/wwww"]
```

`ADD` can be used to add from remote location, as well as extracting from compressed file.

``` dockerfile
# This is a comment
FROM ubuntu:18.10

LABEL maintainer "Me"

RUN yum install apache

ADD https://s3.aws.com/files/my/myfile /var/wwww
ADD my-file.tar.gz /var/wwww
```

Both `COPY` and `ADD` require the source path to be in the context of the build, or the working directory.

## Arguments and environment variables

Argument can be passed in by using `--build-arg key1=value1 key2=value2`.

``` dockerfile
FROM ubuntu:18.10

LABEL maintainer "Me"

ARG MY_VERSION=1.0.0
ENV MY_VERSION ${MY_VERSION}
```

Secrets should not be passed in through `ARG` or `ENV`

## Working directory

Can be supplied by `WORKDIR`

``` dockerfile
FROM ubuntu:18.10

LABEL maintainer "Me"

WORKDIR <dir>
```

## Change user

User/group can be changed by using `USER`

``` dockerfile
FROM ubuntu:18.10

LABEL maintainer "Me"

WORKDIR <dir>
USER <uid>:<gid> or <username>:<group name>
```

## Expose port
`EXPOSE` can be used to expose port from container

``` dockerfile
FROM ubuntu:18.10

LABEL maintainer "Me"

EXPOSE 80,443
```

## Run commands
`CMD`

## Best Practice

### Create ephemeral containers
Similar to 12 factors processes, docker container should be as ephemeral as possible. This means the container can be stopped and destroyed, then rebuilt and replaced with an absolute minimum set up and configuration

### Understand docker context
When issuing a docker build command, the current working directory is called the build context. By default, the Dockerfile is assumed to be located here, but can be specified to use a different location with the file flag (-f). Regardless of where the Dockerfile actually lives, all recursive contents of files and directories in the current directory are sent to the Docker daemon as the build context. Specify a smaller docker context folder helps reducing image size and build time.

``` bash
docker build -t <name:tag> -f <Dockerfile> <context_folder>
```

A remote context is also supported in URL format.

To avoid restructuring source code repository, a `.dockerignore` file can be supplied to exclude files not relevant to the build.

### Leveraging build cache
 if a build contains several layers, they can be ordered from the less frequently changed (to ensure the build cache is reusable) to the more frequently changed:

- Install tools you need to build your application
- Install or update application dependencies
- Generate your application

``` Dockerfile
ROM golang:1.9.2-alpine3.6 AS build

# Install tools required for project
# Run `docker build --no-cache .` to update dependencies
RUN apk add --no-cache git
RUN go get github.com/golang/dep/cmd/dep

# List project dependencies with Gopkg.toml and Gopkg.lock
# These layers are only re-built when Gopkg files are updated
COPY Gopkg.lock Gopkg.toml /go/src/project/
WORKDIR /go/src/project/
# Install library dependencies
RUN dep ensure -vendor-only

# Copy the entire project and build it
# This layer is rebuilt when a file changes in the project directory
COPY . /go/src/project/
RUN go build -o /bin/project

# This results in a single layer image
FROM scratch
COPY --from=build /bin/project /bin/project
ENTRYPOINT ["/bin/project"]
CMD ["--help"]
```

### Decouple applications
Each container should have only one concern. Decoupling applications into multiple containers makes it easier to scale horizontally and reuse containers.

### Minimise the number of layers
Image with multiple layers is large to pull/push and could have performance problem. To reduce layers:

- Consider use docker Docker 1.10 and higher as only the instructions `RUN`, `COPY`, `ADD` create layers
- Use multi-stage build in Docker 17.05 and higher

### Sort multi-line arguments
Whenever possible, ease later changes by sorting multi-line arguments alphanumerically. This helps to avoid duplication of packages and make the list much easier to update.

``` Dockerfile
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```
