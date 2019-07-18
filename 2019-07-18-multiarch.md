# Multiarch

## Condition

Fedora 30

```
$ uname -m
x86_64
```

```
$ rpm -q podman
podman-1.4.2-1.fc30.x86_64

$ rpm -qf /usr/bin/docker
docker-ce-cli-18.09.5-3.fc29.x86_64
```


## podman

```
$ sudo dnf remove qemu-user-static
```

### Fedora

```
$ podman pull arm64v8/fedora

$ podman run --rm -t arm64v8/fedora uname -m
aarch64
```

```
$ skopeo inspect --raw docker://registry.fedoraproject.org/fedora:30 | jq
$ podman pull docker://registry.fedoraproject.org/fedora@sha256:81bc8a5a9b0cc196faba64c311d19fccef5b74a774cd0cade49e8cd2e18aaa25
$ podman run --rm -t docker://registry.fedoraproject.org/fedora@sha256:81bc8a5a9b0cc196faba64c311d19fccef5b74a774cd0cade49e8cd2e18aaa25 uname -m
aarch64
```

### CentOS

```
$ podman pull arm64v8/centos
$ podman run --rm -t arm64v8/centos uname -m
aarch64
```

### RHEL

```
$ skopeo inspect --raw docker://registry.access.redhat.com/ubi8

$ podman pull docker://registry.access.redhat.com/ubi8@sha256:2c448442162d3a4288650ecf63dc41ad1526bc6a830d0af8df507aa4db32faad
$ podman run --rm -t docker://registry.access.redhat.com/ubi8@sha256:2c448442162d3a4288650ecf63dc41ad1526bc6a830d0af8df507aa4db32faad uname -m
aarch64
```

## docker

```
$ docker pull arm64v8/fedora

$ docker run --rm -t arm64v8/fedora uname -m
aarch64
failed to resize tty, using default size
```

## qemu-user-static RPM

qemu-user-static RPM.

```
$ podman system prune -a -f

$ podman images
REPOSITORY   TAG   IMAGE ID   CREATED   SIZE
```

```
$ sudo dnf remove qemu-user-static
$ sudo dnf install qemu-user-static

$ rpm -q qemu-user-static
qemu-user-static-3.1.0-9.fc30.x86_64
```

See https://hub.docker.com/_/fedora/ => https://hub.docker.com/r/arm64v8/fedora/

```
$ podman pull arm64v8/fedora
```

```
$ podman images
REPOSITORY                 TAG      IMAGE ID       CREATED        SIZE
docker.io/arm64v8/fedora   latest   55dc61afd232   19 hours ago   289 MB
```

Why does `podman run --rm -t arm64v8/fedora uname -m` work?

```
$ podman run --rm -t arm64v8/fedora ls /usr/bin/qemu-aarch64-static

$ podman run --rm -t arm64v8/fedora uname -m
aarch64

$ podman run --rm -t arm64v8/fedora ls /usr/bin/qemu-aarch64-static
/usr/bin/ls: cannot access '/usr/bin/qemu-aarch64-static': No such file or directory

$ podman run --rm -t -v /usr/bin/qemu-aarch64-static:/usr/bin/qemu-aarch64-static arm64v8/fedora uname -m
aarch64
```

```
$ sudo dnf remove qemu-user-static
```

```
$ podman run --rm -t arm64v8/fedora uname -m
aarch64
```


## multiarch qemu-user-static + different architecture container image

multiarch qemu-user-static is not qemu-user-static RPM.

multiarch qemu-user-static
https://github.com/multiarch/qemu-user-static

```
$ podman system prune -a -f

$ podman images
REPOSITORY   TAG   IMAGE ID   CREATED   SIZE
```

```
$ sudo dnf install qemu-user-static
```

```
$ rpm -ql qemu-user-static
/usr/bin/qemu-aarch64-static
/usr/bin/qemu-aarch64_be-static
...
/usr/lib/binfmt.d/qemu-aarch64-static.conf
/usr/lib/binfmt.d/qemu-aarch64_be-static.conf
...
```

```
$ podman run --rm --privileged multiarch/qemu-user-static:register --reset
Trying to pull docker.io/multiarch/qemu-user-static:register...Getting image source signatures
Copying blob 2b7cb15a4b93 done
Copying blob 8e674ad76dce done
Copying blob 96aa3d7b9b30 done
Copying blob 8d669bf48302 done
Copying config 20d331b290 done
Writing manifest to image destination
Storing signatures
mount: permission denied (are you root?)
mount: permission denied (are you root?)
```

```
$ podman images
REPOSITORY                             TAG        IMAGE ID       CREATED       SIZE
docker.io/multiarch/qemu-user-static   register   20d331b290b0   2 weeks ago   1.48 MB
```

```
$ sudo podman run --rm --privileged multiarch/qemu-user-static:register --reset
Setting /usr/bin/qemu-alpha-static as binfmt interpreter for alpha
Setting /usr/bin/qemu-arm-static as binfmt interpreter for arm
Setting /usr/bin/qemu-armeb-static as binfmt interpreter for armeb
Setting /usr/bin/qemu-sparc32plus-static as binfmt interpreter for sparc32plus
Setting /usr/bin/qemu-ppc-static as binfmt interpreter for ppc
Setting /usr/bin/qemu-ppc64-static as binfmt interpreter for ppc64
Setting /usr/bin/qemu-ppc64le-static as binfmt interpreter for ppc64le
Setting /usr/bin/qemu-m68k-static as binfmt interpreter for m68k
Setting /usr/bin/qemu-mips-static as binfmt interpreter for mips
Setting /usr/bin/qemu-mipsel-static as binfmt interpreter for mipsel
Setting /usr/bin/qemu-mipsn32-static as binfmt interpreter for mipsn32
Setting /usr/bin/qemu-mipsn32el-static as binfmt interpreter for mipsn32el
Setting /usr/bin/qemu-mips64-static as binfmt interpreter for mips64
Setting /usr/bin/qemu-mips64el-static as binfmt interpreter for mips64el
Setting /usr/bin/qemu-sh4-static as binfmt interpreter for sh4
Setting /usr/bin/qemu-sh4eb-static as binfmt interpreter for sh4eb
Setting /usr/bin/qemu-s390x-static as binfmt interpreter for s390x
Setting /usr/bin/qemu-aarch64-static as binfmt interpreter for aarch64
Setting /usr/bin/qemu-aarch64_be-static as binfmt interpreter for aarch64_be
Setting /usr/bin/qemu-hppa-static as binfmt interpreter for hppa
Setting /usr/bin/qemu-riscv32-static as binfmt interpreter for riscv32
Setting /usr/bin/qemu-riscv64-static as binfmt interpreter for riscv64
Setting /usr/bin/qemu-xtensa-static as binfmt interpreter for xtensa
Setting /usr/bin/qemu-xtensaeb-static as binfmt interpreter for xtensaeb
Setting /usr/bin/qemu-microblaze-static as binfmt interpreter for microblaze
Setting /usr/bin/qemu-microblazeel-static as binfmt interpreter for microblazeel
Setting /usr/bin/qemu-or1k-static as binfmt interpreter for or1k
```

```
$ podman images
REPOSITORY                             TAG        IMAGE ID       CREATED       SIZE
docker.io/multiarch/qemu-user-static   register   20d331b290b0   2 weeks ago   1.48 MB
```

See https://hub.docker.com/_/fedora/ => https://hub.docker.com/r/arm64v8/fedora/

```
$ podman pull arm64v8/fedora
```

```
$ podman images
REPOSITORY                             TAG        IMAGE ID       CREATED        SIZE
docker.io/arm64v8/fedora               latest     55dc61afd232   18 hours ago   289 MB
docker.io/multiarch/qemu-user-static   register   20d331b290b0   2 weeks ago    1.48 MB
```

```
$ podman run --rm -t arm64v8/fedora uname -m
standard_init_linux.go:207: exec user process caused "no such file or directory"
```

```
$ podman run --rm -t -v /usr/bin/qemu-aarch64-static:/usr/bin/qemu-aarch64-static arm64v8/fedora uname -m
aarch64
```

## multiarch qemu-user-static + multiarch compatible images

https://github.com/multiarch/fedora

```
$ podman pull multiarch/fedora:30-aarch64
$ podman run --rm -t multiarch/fedora:30-aarch64 uname -m
aarch64
```

