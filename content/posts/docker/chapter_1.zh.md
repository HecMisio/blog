---
layout: post
title: "容器技术及Docker概述"
description: "简单介绍容器技术和 Docker"
date: 2025-02-20T17:13:06+08:00
image: "/posts/docker/images/chapter_1-cover.png"
tags: ["Docker"]
---

## 虚拟化技术

在讨论容器之前，我们先来了解一下什么是虚拟化技术。在计算服务发展的早期，每个服务器通常只运行一个应用程序或服务，为了让应用能够稳定安全地运行，通常都会为其配备远超性能所需的配置，这导致服务器的资源利用率极低，造成了很大的资源浪费，并且每次要上线新的应用或服务，都需要购置新的物理服务器，也给企业带来了很高的硬件成本和管理成本。

为了解决上述问题，虚拟化技术应运而生。简单来说，虚拟化技术可以帮助用户以单个物理硬件系统为基础，创建多个虚拟运行环境，以提高资源利用率，降低硬件成本和管理成本。

* 1960s-1970s，IBM在大型机上首次引入了虚拟化技术，以支持多用户环境；
* 1990s-2000s，VMWare开创了虚拟机技术，虚拟化技术得以在普通服务器上被应用；
* 2000s-至今，容器技术被广泛应用于大规模分布式系统中，并在 Docker 出现后得以普及。

如今，虚拟化技术已成为现代 IT 基础设施的核心组成部分，是云计算基石，推动了计算模式和大规模分布式系统的发展与变革。

## 什么是容器？

一言以蔽之，容器是一种轻量级、可移植的虚拟化技术，用于打包和运行应用程序及其依赖。

作为虚拟化技术的两大核心，容器和虚拟机经常被拿来对比，它们都被用于整合各种IT组件并与主机系统的其他部分隔离开来，以封装计算环境，二者的区别在于隔离了哪些组件，这反过来又影响了每种方式的规模和可移植性。

<div class="content-image">
<img src="/posts/docker/images/chapter_1-vm-vs-container.png" width="80%">
<p>Virtual Machine vs Container</p>
</div>

* 虚拟机技术（Virtual Machine, VM）：一种通过软件模拟完整计算机系统的技术，允许在单一物理硬件上运行多个独立的操作系统实例。每个虚拟机（VM）都包含自己的虚拟硬件（如 CPU、内存、存储和网络接口），并可以运行独立的操作系统和应用程序。

* 容器技术（Container）：一种轻量级、可移植的虚拟化技术，用于打包和运行应用程序及其依赖。与传统的虚拟机不同，容器共享主机操作系统的内核，但通过隔离机制（如命名空间和控制组）确保应用程序在独立的环境中运行。

<div class="table-container">
  <table>
    <tr><th>特性</th><th>Vistual Machine</th><th>Container</th></tr>
    <tr><th>虚拟化对象</th><td>整个操作系统</td><td>应用程序及其依赖</td></tr>
    <tr><th>启动速度</th><td>分钟级</td><td>秒级</td></tr>
    <tr><th>资源占用</th><td>高（每个虚拟机需要完整操作系统）</td><td>低（共享主机内核）</td></tr>
    <tr><th>隔离性</th><td>完全隔离</td><td>进程级别隔离</td></tr>
    <tr><th>可移植性</th><td>较低（依赖虚拟机镜像）</td><td>高（依赖容器运行时）</td></tr>
  </table>
</div>

## 容器和 Docker 之间的关系

在 Docker 出现之前，容器技术就已经存在，但由于复杂性高、工具链不完善和生态不成熟等原因，应用起来比较困难，因此难以得到广泛使用，直到2013年 Docker 发布，极大简化了容器的使用，推动了容器技术的普及。

现如今 Docker 已经成为容器技术的实际标准，但仍有许多其他容器平台，为了保证不同容器之间能够兼容，Docker、CoreOS、Google等若干公司成立了<i>Open Container Initiative（OCI）</i>来制定容器规范，目前 <i>OCI</i> 发布有两个核心规范，<i>运行时规范（Runtime Specification）</i>和<i>镜像规范（Image Specification）</i>，这两个规范保证了不同组织和厂商开发的容器能够在不同的 runtime 上运行。

## Docker 核心概念

Docker 具有以下核心概念

* Docker Engine：Docker 的核心服务，以C/S架构运行，用于创建和管理 Docker 对象。
* Docker Container：Docker Image 的运行实例，是一个独立的应用程序环境。
* Docker Image：只读模块，runtime 通过 Docker Image 来创建容器。
* Dockerfile：文本文件，包含一系列构建 Docker Image 的指令。
* Registry：存放 Docker Image 的仓库。

<div class="content-image">
<img src="/posts/docker/images/chapter_1-docker-architecture.png" width="80%">
<p>Docker Architecture</p>
</div>

### Docker Engine

Docker Engine是一种用于构建和容器化应用程序的开源容器化技术，以C（Docker CLI）/S（Docker Deamon）模式运行。

#### Docker Deamon

即 Docker Engine 的服务端程序，一般情况下以守护进程（dockerd）的形式运行，提供 Docker APIs 服务，支持创建和管理 Docker 对象，包括镜像（Docker Image）、容器（Docker Container）、网络（Network）和存储卷（Volume）。默认情况下，Docker Deamon 只会响应本地的客户端请求，可以通过配置打开 TCP 监听以支持远程客户端调用。此外，Docker Deamon 还可以与其他 Docker 服务通信，这主要用于构建 Docker 服务集群 Docker Swarm。

#### Docker CLI

即 Docker Engine 的客户端程序，可使用 CLI 命令与 Docker Deamon 交互，调用 Docker APIs，实现对 Docker 对象的管理。

### Docker Container

Docker Container 是一个独立的应用程序进程，它拥有应用程序所依赖的所有文件，从而与主机环境完全隔离。其具有以下特点：

* 自包含：每个 Docker Container 都拥有支持它运行所依赖的所有东西，完全不需要主机预安装任何依赖。

* 隔离性：每个 Docker Container 都是一个隔离沙箱，对主机和其他 Docker Container 的影响极小，提升了应用的安全性。

* 独立性：每个 Docker Container 都是独立的，对一个 Docker Container 的修改不会影响其他 Docker Container。

* 移植性：Docker Container 可以非常便捷地移植到任何运行环境中，无论是在本地还是在服务器上，抑或是在云上。

### Docker Image

Docker Image 是一个标准化的包，它包含了运行一个容器所需要的所有文件、二进制文件、库文件和配置。Docker Image 有两个重要特性：

* Docker Image 是不可修改的，一旦创建，只可以重新构建或以其为基础镜像进行修改。

* Docker Image 是分层结构的，每一层代表了一系列的文件系统修改，增加、删除或修改文件。

### Dockerfile

Dockerfile 是一个文本文件，用于定义如何构建 Docker 镜像。它包含一系列指令，这些指令描述了镜像的基础环境、应用程序代码、依赖项以及运行时配置。通过Dockerfile，可以自动化地构建一致且可重复的 Docker 镜像。

### Registry

Registry 是用于存放镜像并可以与他人分享镜像的仓库，它可以是公开的，也可以是私有的，<a href="https://hub.docker.com/">Docker Hub</a> 是最常用的开放 Registry，你也可以通过如 <a href="https://goharbor.io/">Harbor</a> 这样的应用来构建私有镜像仓库。

## 总结

* 虚拟化技术的出现是为了解决传统计算环境中的资源浪费、成本高、管理复杂等问题。

* 虚拟机和容器都是虚拟化技术，以资源隔离的方式进行来封装计算环境，但因隔离的资源力度不同，导致彼此在虚拟化程度、封装性、隔离性、迁移性上存在较大区别。
<div class="table-container">
  <table>
    <tr><th>特性</th><th>Vistual Machine</th><th>Container</th></tr>
    <tr><th>虚拟化对象</th><td>整个操作系统</td><td>应用程序及其依赖</td></tr>
    <tr><th>启动速度</th><td>分钟级</td><td>秒级</td></tr>
    <tr><th>资源占用</th><td>高（每个虚拟机需要完整操作系统）</td><td>低（共享主机内核）</td></tr>
    <tr><th>隔离性</th><td>完全隔离</td><td>进程级别隔离</td></tr>
    <tr><th>可移植性</th><td>较低（依赖虚拟机镜像）</td><td>高（依赖容器运行时）</td></tr>
  </table>
</div>

* Docker 的出现推动了容器技术的普及，推动了容器技术的标准化进程。Docker 的核心组件 Docker Engine 由 Docker Deamon 和 Docker CLI 组成，Docker CLI 作为客户端，接受用户命令并转发请求给 Docker Deamon，Docker Deamon 接受客户端请求以创建和管理镜像和容器，以及网络和存储卷。Image 是一个标准化的包，且无法修改，包含了运行一个 Container 所需要的所有文件。Container 则是一个 Image 实例，是一个独立的应用程序进程，与主机和其他 Container 完全隔离。Dockerfile 是记录如何构建 Image 的文本文件，可通过它来自动化构建一致且重复的 Image。Registry 则是存放和与他人共享 Image 的仓库。