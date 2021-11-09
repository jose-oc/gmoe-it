---
title: "Docker Useful Commands"
date: 2021-11-09T17:34:34+01:00
draft: true
---

Some useful docker commands.

## Show all images with a label

```sh
docker image ls --all --filter "label=Description=This image is used to run my side project called ABC"
docker images --filter "label=Vendor=MyCompany"
docker images --filter "label=Vendor=MyCompany" --format 'table {{.Repository}}\t{{.Tag}}\t{{.ID}}\t{{.CreatedAt}}\t{{.Size}}'
```



## Remove all images with a label

In this case we filter by a label set in the Dockerfile of the BBC images:

```sh
docker image prune --all --filter "label=Description=This image is used to run my side project called ABC"
```

Then you can *remove all unused volumes*

```sh
docker volume prune
```


## Readonly containers

Docker readonly but writable directories.

Readonly container with `--readonly`:

```shell
docker container run --rm --read-only alpine:3.7 touch /tmp/hello.txt
touch: /tmp/hello.txt: Read-only file system
```

But you can allow writable directories with `tmpfs` type of mount:

```sh
docker container run --rm --read-only --mount type=tmpfs,destination=/tmp --mount type=tmpfs,destination=/run alpine:3.7 touch /tmp/hello.txt /run/bye.txt
```

