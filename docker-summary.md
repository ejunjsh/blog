---
title: docker总结
date: 2018-10-24 22:50:22
tags: [docker]
categories: docker
---
> 不想分几章了，所以很长很长👿

# docker 容器的状态机

[![](http://idiotsky.top/images3/docker-summary-1.jpg)](http://idiotsky.top/images3/docker-summary-1.jpg)

一个容器在某个时刻可能处于以下几种状态之一：

* created：已经被创建 （使用 docker ps -a 命令可以列出）但是还没有被启动 （使用 docker ps 命令还无法列出）
* running：运行中
* paused：容器的进程被暂停了
* restarting：容器的进程正在重启过程中
* exited：上图中的 stopped 状态，表示容器之前运行过但是现在处于停止状态（要区别于 created 状态，它是指一个新创出的尚未运行过的容器）。可以通过 start 命令使其重新进入 running 状态
* destroyed：容器被删除了，再也不存在了

<!-- more -->

# Docker 命令概述

我们可以把Docker 的命令大概地分类如下：

````
镜像操作：
    build     Build an image from a Dockerfile
    commit    Create a new image from a container's changes
    images    List images
    load      Load an image from a tar archive or STDIN
    pull      Pull an image or a repository from a registry
    push      Push an image or a repository to a registry
    rmi       Remove one or more images
    search    Search the Docker Hub for images
    tag       Tag an image into a repository
    save      Save one or more images to a tar archive (streamed to STDOUT by default)
    history   显示某镜像的历史
    inspect   获取镜像的详细信息

    容器及其中应用的生命周期操作：
    create    Create a new container （创建一个容器）        
    kill      Kill one or more running containers
    inspect   Return low-level information on a container, image or task
    pause     Pause all processes within one or more containers
    ps        List containers
    rm        Remove one or more containers （删除一个或者多个容器）
    rename    Rename a container
    restart   Restart a container
    run       Run a command in a new container （创建并启动一个容器）
    start     Start one or more stopped containers （启动一个处于停止状态的容器）
    stats     Display a live stream of container(s) resource usage statistics （显示容器实时的资源消耗信息）
    stop      Stop one or more running containers （停止一个处于运行状态的容器）
    top       Display the running processes of a container
    unpause   Unpause all processes within one or more containers
    update    Update configuration of one or more containers
    wait      Block until a container stops, then print its exit code
    attach    Attach to a running container
    exec      Run a command in a running container
    port      List port mappings or a specific mapping for the container
    logs      获取容器的日志    
    
    容器文件系统操作：
    cp        Copy files/folders between a container and the local filesystem
    diff      Inspect changes on a container's filesystem
    export    Export a container's filesystem as a tar archive
    import    Import the contents from a tarball to create a filesystem image
    
    Docker registry 操作：
    login     Log in to a Docker registry.
    logout    Log out from a Docker registry.
    
    Volume 操作
    volume    Manage Docker volumes
    
    网络操作
    network   Manage Docker networks
    
    Swarm 相关操作
    swarm     Manage Docker Swarm
    service   Manage Docker services
    node      Manage Docker Swarm nodes       
    
    系统操作：    
    version   Show the Docker version information
    events    Get real time events from the server  (持续返回docker 事件)
    info      Display system-wide information （显示Docker 主机系统范围内的信息）
````

# Doker 平台的基本构成

[![](http://idiotsky.top/images3/docker-summary-2.jpg)](http://idiotsky.top/images3/docker-summary-2.jpg)

Docker 平台基本上由三部分组成：

* 客户端：用户使用 Docker 提供的工具（CLI 以及 API 等）来构建，上传镜像并发布命令来创建和启动容器
* Docker 主机：从 Docker registry 上下载镜像并启动容器
* Docker registry：Docker 镜像仓库，用于保存镜像，并提供镜像上传和下载


# Docker 镜像分层,COW 和 镜像大小（size）

## 镜像分层和容器层

一个 Docker 镜像是基于基础镜像的多层叠加，最终构成和容器的 rootfs （根文件系统）。当 Docker 创建一个容器时，它会在基础镜像的容器层之上添加一层新的薄薄的可写容器层。接下来，所有对容器的变化，比如写新的文件，修改已有文件和删除文件，都只会作用在这个容器层之中。因此，通过不拷贝完整的 rootfs，Docker 减少了容器所占用的空间，以及减少了容器启动所需时间。

[![](http://idiotsky.top/images3/docker-summary-3.jpg)](http://idiotsky.top/images3/docker-summary-3.jpg)

## COW 和镜像大小

COW，copy-on-write 技术，一方面带来了容器启动的快捷，另一方也造成了容器镜像大小的增加。每一次 RUN 命令都会在镜像上增加一层，每一层都会占用磁盘空间。举个例子，在 Ubuntu 14.04 基础镜像中运行 RUN apt-get upgrade 会在保留基础层的同时再创建一个新层来放所有新的文件，而不是修改老的文件，因此，新的镜像大小会超过直接在老的文件系统上做更新时的文件大小。因此，为了减少镜像大小起见，所有文件相关的操作，比如删除，释放和移动等，都需要尽可能地放在一个 RUN 指令中进行。

## 使用容器需要避免的一些做法

这篇文章 [10 things to avoid in docker containers](http://developers.redhat.com/blog/2016/02/24/10-things-to-avoid-in-docker-containers/) 列举了一些在使用容器时需要避免的做法，包括：

* 不要在容器中保存数据（Don’t store data in containers）
* 将应用打包到镜像再部署而不是更新到已有容器（Don’t ship your application in two pieces）
* 不要产生过大的镜像 （Don’t create large images）
* 不要使用单层镜像 （Don’t use a single layer image）
* 不要从运行着的容器上产生镜像 （Don’t create images from running containers ）
* 不要只是使用 “latest”标签 （Don’t use only the “latest” tag）
* 不要在容器内运行超过一个的进程 （Don’t run more than one process in a single container ）
* 不要在容器内保存 credentials，而是要从外面通过环境变量传入 （ Don’t store credentials in the image. Use environment variables）
* 不要使用 root 用户跑容器进程（Don’t run processes as a root user ）
* 不要依赖于IP地址，而是要从外面通过环境变量传入 （Don’t rely on IP addresses ）



# Linux namespace 的概念

Linux 内核从版本 2.4.19 开始陆续引入了 namespace 的概念。其目的是将某个特定的全局系统资源（global system resource）通过抽象方法使得namespace 中的进程看起来拥有它们自己的隔离的全局系统资源实例（The purpose of each namespace is to wrap a particular global system resource in an abstraction that makes it appear to the processes within the namespace that they have their own isolated instance of the global resource. ）。Linux 内核中实现了六种 namespace，按照引入的先后顺序，列表如下：


namespace	|引入的相关内核版本	|被隔离的全局系统资源	|在容器语境下的隔离效果
-------|-----------|-----------|----------------
Mount namespaces|	Linux 2.4.19|	文件系统挂接点	|每个容器能看到不同的文件系统层次结构
UTS namespaces|	Linux 2.6.19|	nodename 和 domainname	|每个容器可以有自己的 hostname 和 domainame
IPC namespaces|	Linux 2.6.19|	特定的进程间通信资源，包括System V IPC 和  POSIX message queues	|每个容器有其自己的 System V IPC 和 POSIX 消息队列文件系统，因此，只有在同一个 IPC namespace 的进程之间才能互相通信
PID namespaces|	Linux 2.6.24	|进程 ID 数字空间 （process ID number space）	|每个 PID namespace 中的进程可以有其独立的 PID； 每个容器可以有其 PID 为 1 的root 进程；也使得容器可以在不同的 host 之间迁移，因为 namespace 中的进程 ID 和 host 无关了。这也使得容器中的每个进程有两个PID：容器中的 PID 和 host 上的 PID。
Network namespaces|	始于Linux 2.6.24 完成于 Linux 2.6.29|	网络相关的系统资源	|每个容器用有其独立的网络设备，IP 地址，IP 路由表，/proc/net 目录，端口号等等。这也使得一个 host 上多个容器内的同一个应用都绑定到各自容器的 80 端口上。
User namespaces	|始于 Linux 2.6.23 完成于 Linux 3.8)|	用户和组 ID 空间|	 在 user namespace 中的进程的用户和组 ID 可以和在 host 上不同； 每个 container 可以有不同的 user 和 group id；一个 host 上的非特权用户可以成为 user namespace 中的特权用户；

Linux namespace 的概念说简单也简单说复杂也复杂。简单来说，我们只要知道，处于某个 namespace 中的进程，能看到独立的它自己的隔离的某些特定系统资源；复杂来说，可以去看看 Linux 内核中实现 namespace 的原理，网络上也有大量的文档供参考，这里不再赘述。


# Docker 容器使用 linux namespace 做运行环境隔离

当 Docker 创建一个容器时，它会创建新的以上六种 namespace 的实例，然后把容器中的所有进程放到这些 namespace 之中，使得Docker 容器中的进程只能看到隔离的系统资源。 

## PID namespace

我们能看到同一个进程，在容器内外的 PID 是不同的：

* 在容器内 PID 是 1，PPID 是 0。
* 在容器外 PID 是 2198， PPID 是 2179 即 docker-containerd-shim 进程.

关于 containerd，containerd-shim 和 container 的关系，[文章](https://github.com/crosbymichael/dockercon-2016/blob/master/Creating%20Containerd.pdf) 中的下图可以说明：

[![](http://idiotsky.top/images3/docker-summary-3.jpg)](http://idiotsky.top/images3/docker-summary-3.jpg)

* Docker 引擎管理着镜像，然后移交给 containerd 运行，containerd 再使用 runC 运行容器。
* Containerd 是一个简单的守护进程，它可以使用 runC 管理容器，使用 gRPC 暴露容器的其他功能。它管理容器的开始，停止，暂停和销毁。由于容器运行时是孤立的引擎，引擎最终能够启动和升级而无需重新启动容器。
* runC是一个轻量级的工具，它是用来运行容器的，只用来做这一件事，并且这一件事要做好。runC基本上是一个小命令行工具且它可以不用通过Docker引擎，直接就可以使用容器。
  
因此，容器中的主应用在 host 上的父进程是 containerd-shim，是它通过工具 runC 来启动这些进程的。

这也能看出来，pid namespace 通过将 host 上 PID 映射为容器内的 PID， 使得容器内的进程看起来有个独立的 PID 空间。


## UTS namespace

类似地，容器可以有自己的 hostname 和 domainname：


> to be continue ...


# 参考

http://www.cnblogs.com/sammyliu/p/5875470.html

http://www.cnblogs.com/sammyliu/p/5878973.html

