---
layout: post
title: "构建 Docker Image 的最佳姿势"
description: "介绍 Docker Image 的内部结构，讲解构建 Docker Image 的最佳实践"
date: 2025-02-23T15:25:35+08:00
image: "/posts/docker/images/chapter_2-cover.jpg"
tags: ["Docker"]
---

## 镜像内部结构

### Bootfs 和 Rootfs

在介绍 Docker Image 结构之前，首先了解一下 Linux 的文件系统。Linux 的文件系统可以分为 Bootfs 和 Rootfs 两个部分。

* Bootfs 是引导文件系统，主要包含引导加载程序（Bootloader）和系统内核（Kernal），主要作用是在系统启动时加载内核到内存，一旦内核加载完成，Bootfs 就会被卸载，因此它是一个只读文件系统。

* Rootfs 是根文件系统，也是用户文件系统。它包含了系统运行所需的目录结构和文件，如 `/etc`，`/bin`，`/usr` 等，是用户可修改的部分。

我们总说 Docker 容器之间共享操作系统内核，其实共享的就是 Bootfs，彼此之间的环境隔离，隔离的就是 Rootfs。

<div class="content-image">
<img src="/posts/docker/images/chapter_2-bootfs-and-rootfs.png" width="60%">
<p>Bootfs and Rootfs</p>
</div>

### 镜像的分层结构

由上文可知，构建一个 Docker Image，就是构建 Rootfs 的过程，Docker 采用联合文件系统（Union File System）的方式来构建 Docker Image。联合文件系统是一种分层文件系统，它将文件系统分为多层，每一层可以独立管理，但对外呈现为单一的文件系统。我们可以使用 `docker history` 命令来查看一个镜像文件的分层结构。

<div class="content-image">
<img src="/posts/docker/images/chapter_2-image-history.png" width="100%">
<p>Image History</p>
</div>

联合文件系统有一个重要特征是写时复制（Copy-on-Write），当需要修改只读层中的数据时，实际操作是在可写层创建新副本并进行修改的，从而保持原始数据不变。对于 Docker 来说，Docker Image 就是只读层，而可写层则是 Docker Container，因此 Docker Container 实际上存储了对 Docker Image 的修改。

## 镜像构建

### 基础镜像

通常来说我们希望镜像能够提供基本的操作系统环境，用户可以根据需要安装和配置其他软件或文件，能够满足这样的需求的镜像就可以称为基础镜像。常用的基础镜像一般都是各种 Linux 发行版的镜像，scratch 镜像（特殊镜像，里面什么也没有）使用较少。

一般可通过两种方式来构建镜像，一种是通过 `docker commit` 命令将正在运行的容器保存为镜像，一种则是通过 Dockerfile 和 `docker build` 命令来构建镜像。

### docker commit

`docker commit` 基于现有容器的修改生成新的镜像。它捕获容器当前状态，包括文件系统更改，但不会记录构建过程。这种方式虽然简单便捷，但是缺乏可重复性，难以追踪具体改动，不易维护，因此更适用于临时或简单的场景。

<div class="content-image">
<img src="/posts/docker/images/chapter_2-docker-commit.png" width="100%">
<p>Docker Commit</p>
</div>

### docker build

`docker build` 通过执行 Dockerfile 中定义的指令来构建镜像，提供高度可控的自动化流程，支持复杂的构建逻辑和优化，这也是在生产环境和需要长期维护的项目中比较推荐的做法。

<div class="content-image">
<img src="/posts/docker/images/chapter_2-docker-build.png" width="100%">
<p>Docker Build</p>
</div>

## Dockerfile

从 docker build 小节中的例子中可以看到，Dockerfile 是由一组 `[INSTRUCTION arguments]` 指令构成的文本文件，执行 `docker build` 命令就会按照该文件的描述来构建镜像。

Dockerfile 的第一行指令 `FROM` 将指定基础镜像，随后的每一行指令都会为该镜像添加一层文件，就这样一层层叠加上去，形成最终的镜像文件。Dockerfile 指令可以参考 <a href="https://docs.docker.com/reference/dockerfile/">Dockerfile reference</a>。其中有两组指令容易混淆，这里做特殊说明。

### ADD vs COPY

`ADD` 和 `COPY` 指令都用于将文件或目录从主机复制到镜像中，但它们有一些关键区别。

`ADD` 命令除了复制本地文件或目录，还可以从远程 URL 下载文件并添加到镜像中。如果源文件是压缩包，`ADD` 还会自动解压。

```Dockerfile
ADD http://example.com/file.tar.gz /app/

ADD local-file.tar.gz /app/
```

`COPY` 命令则仅能用于本地文件或目录，不支持从 URL 下载或自动解压。

```Dockerfile
COPY local-file.txt /app/

COPY local-dir/ /app/
```

官方推荐优先使用 `COPY`，除非需要用到 `ADD` 的特定功能，以保持 Dockerfile 的清晰和可预测性。

### RUN vs CMD vs ENTRYPOINT

`RUN`、`CMD` 和 `ENTRYPOINT` 都是用于定义容器行为的指令，但在作用和使用场景上有所不同。

这三个指令都支持 exec 写法和 shell 写法，这两种写法的区别在于 shell 写法会在命令前默认加上 `/bin/sh -c`，对命令进行 shell 解析，并且可以通过 `SHELL` 指令更换默认的 shell。`RUN` 常用 shell 写法，而 `CMD` 和 `ENTRYPOINT` 则推荐使用 exec 写法，除非需要依赖 shell。

`RUN` 用于在构建镜像时执行命令，并将结果保存到镜像中，常用于安装软件。实际上 `RUN` 命令会在构建镜像过程中创建临时容器，并将该容器通过 `docker commit` 保存为中间镜像。

```Dockerfile
RUN apt-get update && apt-get install -y python3
```

`CMD` 和 `ENTRYPOINT` 则是用于在镜像构建后，容器启动时的要执行的命令。不同点在于 `CMD` 定义的是容器的默认执行命令，可以被 `docker run` 命令后指定的命令覆盖，通常用于定义容器启动后的默认行为。

```Dockerfile
CMD ["python", "app.py"]

CMD echo "Hello, World!"
```

而 `ENTRYPOINT` 则定义的是容器的主命令，不会被覆盖，通常被用于将容器包装成可执行程序，并且 `ENTRYPOINT` 和 `CMD` 可以配合使用，将 `CMD` 作为 `ENTRYPOINT` 的默认参数，并且可以在 `docker run` 命令后替换。

```Dockerfile
ENTRYPOINT ["python", "app.py"]

ENTRYPOINT echo "Hello, World!"
```
```Dockerfile
ENTRYPOINT ["python"]

CMD ["app.py"]

docker run my-image # python app.py
docker run my-image script.py # python script.py
```
### 镜像缓存

在 Docker 中，镜像缓存是一种优化机制，用于加速镜像构建过程。Docker 会缓存每一层的构建结果，如果后续构建时的上下文和指令没有发生变化，Docker 会直接使用缓存的结果，而不是重新执行指令，在拉取镜像时也有体现。但如果某条指令的上下文发生变化，缓存会从该指令开始失效，后续的所有指令都会重新执行。因此善用镜像缓存是加速构建镜像的重要优化机制，通过合理安排指令顺序、最小化上下文、合并指令等方法，可以最大化利用缓存。当然也可以在 `docker build` 后使用 `--no-cahce` 参数来强制禁用缓存。

## 最佳实践

### 镜像构建过程优化

优化镜像构建过程可以提高构建速度、收缩镜像大小，提高开发效率。以下提供一些常见的优化方法。

* 使用 `.dockerignore` 文件，收缩构建上下文的大小。

* 合理安排 Dockerfile 指令顺序，将变化频率低的指令放在前面，以充分利用缓存。

* 合并 `RUN`、`CMD` 指令，减少镜像层数，并注意及时清理不需要的文件，收缩镜像大小。

```Dockerfile
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```
```Dockerfile
# 未优化
COPY file1.txt /app/
COPY file2.txt /app/

# 优化后
COPY file1.txt file2.txt /app/
----------------------------------
COPY *.txt /app/
----------------------------------
COPY my-dir/ /app/
```
```Dockerfile
# 未优化
COPY script.sh /app/
RUN chmod +x /app/script.sh

# 优化后
COPY script.sh /app/ && \
    chmod +x /app/script.sh
```

* 选择更加轻量化的基础镜像。

* 使用多阶段构建，收缩最终镜像的大小。

* 使用 `ARG` 和 `ENV` 优化构建。使用 `ARG` 传递构建时的变量，使用 `ENV` 设置运行时环境变量。

```Dockerfile
ARG APP_VERSION=1.0
ENV APP_VERSION=${APP_VERSION}
```

### 镜像 Tag

镜像 tag 是镜像名的重要组成部分，善用 tag 能够帮助我们更好的管理镜像版本。

首先需要注意 latest tag，latest 并没有什么特殊含义，只是当没指明镜像 tag 时，默认使用 latest 而已。因此当我们使用镜像时，最好明确指明某个 tag。

另外 Docker 社区普遍使用多个 tag 对应同一镜像的方案。如当发布 myimage 镜像时，给镜像打上 1.1.0、1.1、1 和 latest 4个 tag，当 1.1.1 发布时，也同时发布 1.1.1、1.1、1 和 latest 4个 tag，如此可以保证 latest 永远是最新版本，而 1.1 和 1 永远是各自分支中最新的版本，如果想要使用特定版本的镜像，则可以通过 x.x.x
 来指定。

## 总结

* 容器间共享 Bootfs 并彼此隔离 Rootfs 来保证应用运行环境的隔离性。Docker Image 使用 UnionFS 实现分层架构，并利用 Copy-on-Write 机制，来保持镜像层只读，在容器层保存对镜像的修改。

* `docker commit` 和 `docker build` 都可用于构建镜像，但 `docker commit` 仅适用于临时、简单的场景下，在生产情况下还是要使用 `docker build` 和 Dockerfile 来构建镜像。

* Dockerfile 指令中，`ADD` 和 `COPY` 指令虽然都可以将本机文件或目录复制到容器内，但 `ADD` 指令还可以实现远程文件的复制和压缩文件的自动解压缩，一般情况下使用 `COPY` 指令即可；`RUN`、`CMD` 和 `ENTRYPOINT` 指令虽然都用于执行命令，但 `RUN` 用于在构建过程中执行，常用于安装软件和配置环境，而 `CMD` 和 `ENTRYPOINT` 则是用于定义容器启动时的动作，并且 `CMD` 用于设置默认执行命令，可以被覆盖，而 `ENTRYPOINT` 用于设置主命令，无法被覆盖，二者可以结合使用，将 `CMD` 作为 `ENTRYPOINT` 的默认参数。

* 优化镜像构建过程，可以加快构建速度，收缩镜像大小，可以从减少构建上下文文件大小和优化 Dockerfile 指令两个方面考虑。

* 合理地为镜像设置 tag 有助于镜像版本管理，通常使用多 tag 指向同一镜像的方案，即发布时同时为镜像打上 x.x.x，x.x，x 和 latest tag。