# arm-docker-build-example
How to build docker images for arm archtecture (e.g. Raspberry Pi) example.

## What's archtecture?
There are several archtecture types in CPU.

For example, x86_64, amd64, arm and i386.

Most people's computers are x86_64 archtecture.

Raspberry Pi is arm32v7 archtecture.

## Effect of archtecture differences
Docker images for other archtectures do not work.

Here are some examples.

exec x86_64 image on x86_64 archtecture: **OK**

```console
# exec on x86_64 archtecture
$ docker run --rm node:12-alpine node -v
v12.15.0
```

exec x86_64 image on arm32v7 archtecture: **OK** (depends on image)

```console
# exec on arm32v7 archtecture (Raspberry Pi)
$ docker run --rm node:12-alpine node -v
v12.15.0
```

exec arm32v7 image on x86_64 archtecture: **NG**

```console
# exec on x86_64 archtecture
$ docker run --rm arm32v7/node:12-alpine node -v
standard_init_linux.go:211: exec user process caused "exec format error"
```

exec arm32v7 image on arm32v7 archtecture: **OK**

```console
# exec on arm32v7 archtecture (Raspberry Pi)
# docker run --rm arm32v7/node:12-alpine node -v
v12.15.0
```

## Problem
**How to build docker images for arm32 with CI Service?**

Most of CI services are x86_64 archtecture based.

Travis CI supports arm64 archtecture, but Raspberry Pi is arm32.

**Answer: use [multiarch/qemu-user-static](https://github.com/multiarch/qemu-user-static)**

### GitHub Actions example

```yaml
name: main

on: [push, pull_request]

jobs:
  main:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: setup docker for arm
        run: docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
      - name: build image
        run: docker build -f Dockerfile.arm -t owner/name:arm .
      - name: push image
        run: |
          docker login -u ${{ secrets.DOCKER_USER }} -p ${{ secrets.DOCKER_PASS }}
          docker push owner/name:arm
```
