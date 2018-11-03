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

````
root@devstack:/home/sammy# ps -ef | grep python
root 2198 2179 0 00:06 ? 00:00:00 python app.py

root@devstack:/home/sammy# ps -ef | grep 2179
root 2179 765 0 00:06 ? 00:00:00 docker-containerd-shim 8b7dd09fbcae00373207f01e2acde45740871c9e3b98286b5458b4ea09f41b3e /var/run/docker/libcontainerd/8b7dd09fbcae00373207f01e2acde45740871c9e3b98286b5458b4ea09f41b3e docker-runc
root 2198 2179 0 00:06 ? 00:00:00 python app.py
root 2249 1692 0 00:06 pts/0 00:00:00 grep --color=auto 2179


root@devstack:/home/sammy# docker exec -it web31 ps -ef
UID PID PPID C STIME TTY TIME CMD
root 1 0 0 16:06 ? 00:00:00 python app.py
````

关于 containerd，containerd-shim 和 container 的关系，[文章](https://github.com/crosbymichael/dockercon-2016/blob/master/Creating%20Containerd.pdf) 中的下图可以说明：

[![](http://idiotsky.top/images3/docker-summary-3.jpg)](http://idiotsky.top/images3/docker-summary-3.jpg)

* Docker 引擎管理着镜像，然后移交给 containerd 运行，containerd 再使用 runC 运行容器。
* Containerd 是一个简单的守护进程，它可以使用 runC 管理容器，使用 gRPC 暴露容器的其他功能。它管理容器的开始，停止，暂停和销毁。由于容器运行时是孤立的引擎，引擎最终能够启动和升级而无需重新启动容器。
* runC是一个轻量级的工具，它是用来运行容器的，只用来做这一件事，并且这一件事要做好。runC基本上是一个小命令行工具且它可以不用通过Docker引擎，直接就可以使用容器。
  
因此，容器中的主应用在 host 上的父进程是 containerd-shim，是它通过工具 runC 来启动这些进程的。

这也能看出来，pid namespace 通过将 host 上 PID 映射为容器内的 PID， 使得容器内的进程看起来有个独立的 PID 空间。


## UTS namespace

类似地，容器可以有自己的 hostname 和 domainname：

````
root@devstack:/home/sammy# hostname
devstack
root@devstack:/home/sammy# docker exec -it web31 hostname
8b7dd09fbcae
````

## user namespace

在 Docker 1.10 版本之前，Docker 是不支持 user namespace。也就是说，默认地，容器内的进程的运行用户就是 host 上的 root 用户，这样的话，当 host 上的文件或者目录作为 volume 被映射到容器以后，容器内的进程其实是有 root 的几乎所有权限去修改这些 host 上的目录的，这会有很大的安全问题。

举例：

* 启动一个容器： docker run -d -v /bin:/host/bin --name web34 training/webapp python app.py
* 此时进程的用户在容器内和外都是root，它在容器内可以对 host 上的 /bin 目录做任意修改

而 Docker 1.10 中引入的 user namespace 就可以让容器有一个 “假”的  root 用户，它在容器内是 root，在容器外是一个非 root 用户。也就是说，user namespace 实现了 host users 和 container users 之间的映射。

启用步骤：

1. 修改 /etc/default/docker 文件，添加行  DOCKER_OPTS="--userns-remap=default"
2. 重启 docker 服务，此时 dockerd 进程为 /usr/bin/dockerd --userns-remap=default --raw-logs
3. 然后创建一个容器：docker run -d -v /bin:/host/bin --name web35 training/webapp python app.py
4. 查看进程在容器内外的用户：

````
root@devstack:/home/sammy# ps -ef | grep python
231072    1726  1686  0 01:44 ?        00:00:00 python app.py

root@devstack:/home/sammy# docker exec web35 ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 17:44 ?        00:00:00 python app.py
````
查看文件/etc/subuid 和 /etc/subgid，可以看到 dockermap 用户在host 上的 uid 和 gid 都是 231072：

````
root@devstack:/home/sammy# cat /etc/subuid
sammy:100000:65536
stack:165536:65536
dockremap:231072:65536
root@devstack:/home/sammy# cat /etc/subgid
sammy:100000:65536
stack:165536:65536
dockremap:231072:65536
````

再看文件/proc/1726/uid_map，它表示了容器内外用户的映射关系，即将host 上的 231072 用户映射为容器内的 0 （即root）用户。

````
root@devstack:/home/sammy# cat /proc/1726/uid_map
         0     231072      65536
````

现在，我们试图在容器内修改 host 上的 /bin 文件夹，就会提示权限不足了：

````
root@80993d821f7b:/host/bin# touch test2
touch: cannot touch 'test2': Permission denied
````

这说明通过使用 user namespace，我们就成功地限制了容器内进程的权限。

## network namespace

默认情况下，当 docker 实例被创建出来后，使用 ip netns  命令无法看到容器实例对应的 network namespace。这是因为 ip netns 命令是从 /var/run/netns 文件夹中读取内容的。

步骤：

1. 找到容器的主进程 ID
    ````
    root@devstack:/home/sammy# docker inspect --format '{{.State.Pid}}' web5
    2704
    ````
2. 创建  /var/run/netns 目录以及符号连接
    ````
    root@devstack:/home/sammy# mkdir /var/run/netns
    root@devstack:/home/sammy# ln -s /proc/2704/ns/net /var/run/netns/web5
    ````
3. 此时可以使用 ip netns 命令了
    ````
    root@devstack:/home/sammy# ip netns
    web5
    root@devstack:/home/sammy# ip netns exec web5 ip addr
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
    valid_lft forever preferred_lft forever
    15: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 scope global eth0
    valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:3/64 scope link
    valid_lft forever preferred_lft forever
    ````

# Linux control groups

## 概念

Linux Cgroup 可​​​让​​​您​​​为​​​系​​​统​​​中​​​所​​​运​​​行​​​任​​​务​​​（进​​​程​​​）的​​​用​​​户​​​定​​​义​​​组​​​群​​​分​​​配​​​资​​​源​​​ — 比​​​如​​​ CPU 时​​​间​​​、​​​系​​​统​​​内​​​存​​​、​​​网​​​络​​​带​​​宽​​​或​​​者​​​这​​​些​​​资​​​源​​​的​​​组​​​合​​​。​​​您​​​可​​​以​​​监​​​控​​​您​​​配​​​置​​​的​​​ cgroup，拒​​​绝cgroup 访​​​问​​​某​​​些​​​资​​​源​​​，甚​​​至​​​在​​​运​​​行​​​的​​​系​​​统​​​中​​​动​​​态​​​配​​​置​​​您​​​的​​​ cgroup。所以，可以将 controll groups 理解为 controller （system resource） （for） （process）groups，也就是是说它以一组进程为目标进行系统资源分配和控制。

它主要提供了如下功能： 

* Resource limitation: 限制资源使用，比如内存使用上限以及文件系统的缓存限制。
* Prioritization: 优先级控制，比如：CPU利用和磁盘IO吞吐。
* Accounting: 一些审计或一些统计，主要目的是为了计费。
* Control: 挂起进程，恢复执行进程。

使​​​用​​​ cgroup，系​​​统​​​管​​​理​​​员​​​可​​​更​​​具​​​体​​​地​​​控​​​制​​​对​​​系​​​统​​​资​​​源​​​的​​​分​​​配​​​、​​​优​​​先​​​顺​​​序​​​、​​​拒​​​绝​​​、​​​管​​​理​​​和​​​监​​​控​​​。​​​可​​​更​​​好​​​地​​​根​​​据​​​任​​​务​​​和​​​用​​​户​​​分​​​配​​​硬​​​件​​​资​​​源​​​，提​​​高​​​总​​​体​​​效​​​率​​​。

在实践中，系统管理员一般会利用CGroup做下面这些事（有点像为某个虚拟机分配资源似的）：

* 隔离一个进程集合（比如：nginx的所有进程），并限制他们所消费的资源，比如绑定CPU的核。
* 为这组进程分配其足够使用的内存
* 为这组进程分配相应的网络带宽和磁盘存储限制
* 限制访问某些设备（通过设置设备的白名单）

Linux 系统中，一切皆文件。Linux 也将 cgroups 实现成了文件系统，方便用户使用。

````
root@devstack:/home/sammy# mount -t cgroup
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,relatime,cpuset)
cgroup on /sys/fs/cgroup/cpu type cgroup (rw,relatime,cpu)
systemd on /sys/fs/cgroup/systemd type cgroup (rw,noexec,nosuid,nodev,none,name=systemd)

root@devstack:/home/sammy# lssubsys -m
cpuset /sys/fs/cgroup/cpuset
cpu /sys/fs/cgroup/cpu
cpuacct /sys/fs/cgroup/cpuacct
memory /sys/fs/cgroup/memory
devices /sys/fs/cgroup/devices
freezer /sys/fs/cgroup/freezer
blkio /sys/fs/cgroup/blkio
perf_event /sys/fs/cgroup/perf_event
hugetlb /sys/fs/cgroup/hugetlb

root@devstack:/home/sammy# ls /sys/fs/cgroup/ -l
total 0
drwxr-xr-x 3 root root 0 Sep 18 21:46 blkio
drwxr-xr-x 3 root root 0 Sep 18 21:46 cpu
drwxr-xr-x 3 root root 0 Sep 18 21:46 cpuacct
drwxr-xr-x 3 root root 0 Sep 18 21:46 cpuset
drwxr-xr-x 3 root root 0 Sep 18 21:46 devices
drwxr-xr-x 3 root root 0 Sep 18 21:46 freezer
drwxr-xr-x 3 root root 0 Sep 18 21:46 hugetlb
drwxr-xr-x 3 root root 0 Sep 18 21:46 memory
drwxr-xr-x 3 root root 0 Sep 18 21:46 perf_event
drwxr-xr-x 3 root root 0 Sep 18 21:46 systemd
````

我们看到 /sys/fs/cgroup 目录中有若干个子目录，我们可以认为这些都是受 cgroups 控制的资源以及这些资源的信息。

* blkio — 这​​​个​​​子​​​系​​​统​​​为​​​块​​​设​​​备​​​设​​​定​​​输​​​入​​​/输​​​出​​​限​​​制​​​，比​​​如​​​物​​​理​​​设​​​备​​​（磁​​​盘​​​，固​​​态​​​硬​​​盘​​​，USB 等​​​等​​​）。
* cpu — 这​​​个​​​子​​​系​​​统​​​使​​​用​​​调​​​度​​​程​​​序​​​提​​​供​​​对​​​ CPU 的​​​ cgroup 任​​​务​​​访​​​问​​​。​​​
* cpuacct — 这​​​个​​​子​​​系​​​统​​​自​​​动​​​生​​​成​​​ cgroup 中​​​任​​​务​​​所​​​使​​​用​​​的​​​ CPU 报​​​告​​​。​​​
* cpuset — 这​​​个​​​子​​​系​​​统​​​为​​​ cgroup 中​​​的​​​任​​​务​​​分​​​配​​​独​​​立​​​ CPU（在​​​多​​​核​​​系​​​统​​​）和​​​内​​​存​​​节​​​点。
* devices — 这​​​个​​​子​​​系​​​统​​​可​​​允​​​许​​​或​​​者​​​拒​​​绝​​​ cgroup 中​​​的​​​任​​​务​​​访​​​问​​​设​​​备​​​。​​​
* freezer — 这​​​个​​​子​​​系​​​统​​​挂​​​起​​​或​​​者​​​恢​​​复​​​ cgroup 中​​​的​​​任​​​务​​​。​​​
* memory — 这​​​个​​​子​​​系​​​统​​​设​​​定​​​ cgroup 中​​​任​​​务​​​使​​​用​​​的​​​内​​​存​​​限​​​制​​​，并​​​自​​​动​​​生​​​成​​​​​内​​​存​​​资​源使用​​​报​​​告​​​。​​​
* net_cls — 这​​​个​​​子​​​系​​​统​​​使​​​用​​​等​​​级​​​识​​​别​​​符​​​（classid）标​​​记​​​网​​​络​​​数​​​据​​​包​​​，可​​​允​​​许​​​ Linux 流量​​​控​​​制​​​程​​​序​​​（tc）识​​​别​​​从​​​具​​​体​​​ cgroup 中​​​生​​​成​​​的​​​数​​​据​​​包​​​。​​​
* net_prio — 这个子系统用来设计网络流量的优先级
* hugetlb — 这个子系统主要针对于HugeTLB系统进行限制，这是一个大页文件系统。


默认的话，在 Ubuntu 系统中，你可能看不到 net_cls 和 net_prio 目录，它们需要你手工做 mount：

````
root@devstack:/sys/fs/cgroup# modprobe cls_cgroup
root@devstack:/sys/fs/cgroup# mkdir net_cls
root@devstack:/sys/fs/cgroup# mount -t cgroup -o net_cls none net_cls

root@devstack:/sys/fs/cgroup# modprobe netprio_cgroup
root@devstack:/sys/fs/cgroup# mkdir net_prio
root@devstack:/sys/fs/cgroup# mount -t cgroup -o net_prio none net_prio

root@devstack:/sys/fs/cgroup# ls net_prio/cgroup.clone_children  cgroup.procs          net_prio.ifpriomap  notify_on_release  tasks
cgroup.event_control   cgroup.sane_behavior  net_prio.prioidx    release_agent
root@devstack:/sys/fs/cgroup# ls net_cls/
cgroup.clone_children  cgroup.event_control  cgroup.procs  cgroup.sane_behavior  net_cls.classid  notify_on_release  release_agent  tasks
````

## 实验

### 通过 cgroups 限制进程的 CPU

写一段最简单的 C 程序：

````c
int main(void)
{
    int i = 0;
    for(;;) i++;
    return 0;
}
````

编译，运行，发现它占用的 CPU 几乎到了 100%：

````
 PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 2204 root      20   0    4188    356    276 R 99.6  0.0   0:46.03 hello
````

接下来我们做如下操作：

````
root@devstack:/home/sammy/c# mkdir /sys/fs/cgroup/cpu/hello
root@devstack:/home/sammy/c# cd /sys/fs/cgroup/cpu/hello
root@devstack:/sys/fs/cgroup/cpu/hello# ls
cgroup.clone_children  cgroup.procs       cpu.cfs_quota_us  cpu.stat           tasks
cgroup.event_control   cpu.cfs_period_us  cpu.shares        notify_on_release
root@devstack:/sys/fs/cgroup/cpu/hello# cat cpu.cfs_quota_us
-1
root@devstack:/sys/fs/cgroup/cpu/hello# echo 20000 > cpu.cfs_quota_us
root@devstack:/sys/fs/cgroup/cpu/hello# cat cpu.cfs_quota_us
20000
root@devstack:/sys/fs/cgroup/cpu/hello# echo 2428 > tasks
````

然后再来看看这个进程的 CPU 占用情况：

````
 PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 2428 root      20   0    4188    356    276 R 19.9  0.0   0:46.03 hello
````

它占用的 CPU 几乎就是 20%，也就是我们预设的阈值。这说明我们通过上面的步骤，成功地将这个进程运行所占用的 CPU 资源限制在某个阈值之内了。

如果此时再启动另一个 hello 进程并将其 id 加入 tasks 文件，则两个进程会共享设定的 CPU 限制：

````
  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 2428 root      20   0    4188    356    276 R 10.0  0.0 285:39.54 hello
12526 root      20   0    4188    356    276 R 10.0  0.0   0:25.09 hello
````

### 通过 cgroups 限制进程的 Memory

同样地，我们针对它占用的内存做如下操作：

````
root@devstack:/sys/fs/cgroup/memory# mkdir hello
root@devstack:/sys/fs/cgroup/memory# cd hello/
root@devstack:/sys/fs/cgroup/memory/hello# cat memory.limit_in_bytes
18446744073709551615
root@devstack:/sys/fs/cgroup/memory/hello# echo 64k > memory.limit_in_bytes
root@devstack:/sys/fs/cgroup/memory/hello# echo 2428 > tasks
root@devstack:/sys/fs/cgroup/memory/hello#
````

上面的步骤会把进程 2428 说占用的内存阈值设置为 64K。超过的话，它会被杀掉。


### 限制进程的 I/O

运行命令：

````
sudo dd if=/dev/sda1 of=/dev/null
````

通过 iotop 命令看 IO （此时磁盘在快速转动），此时其写速度为 242M/s：

````
 TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
 2555 be/4 root      242.60 M/s    0.00 B/s  0.00 % 61.66 % dd if=/dev/sda1 of=/dev/null
````

接着做下面的操作：

````
root@devstack:/home/sammy# mkdir /sys/fs/cgroup/blkio/io
root@devstack:/home/sammy# cd /sys/fs/cgroup/blkio/io
root@devstack:/sys/fs/cgroup/blkio/io# ls -l /dev/sda1
brw-rw---- 1 root disk 8, 1 Sep 18 21:46 /dev/sda1
root@devstack:/sys/fs/cgroup/blkio/io# echo '8:0 1048576'  > /sys/fs/cgroup/blkio/io/blkio.throttle.read_bps_device
root@devstack:/sys/fs/cgroup/blkio/io# echo 2725 > /sys/fs/cgroup/blkio/io/tasks
````

结果，这个进程的IO 速度就被限制在 1Mb/s 之内了：

````
 TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
 2555 be/4 root      990.44 K/s    0.00 B/s  0.00 % 96.29 % dd if=/dev/sda1 of=/dev/null
````

## 术语

cgroups 的术语包括：

* 任务（Tasks）：就是系统的一个进程。
* 控制组（Control Group）：一组按照某种标准划分的进程，比如官方文档中的Professor和Student，或是WWW和System之类的，其表示了某进程组。Cgroups中的资源控制都是以控制组为单位实现。一个进程可以加入到某个控制组。而资源的限制是定义在这个组上，就像上面示例中我用的 hello 一样。简单点说，cgroup的呈现就是一个目录带一系列的可配置文件。
* 层级（Hierarchy）：控制组可以组织成hierarchical的形式，既一颗控制组的树（目录结构）。控制组树上的子节点继承父结点的属性。简单点说，hierarchy就是在一个或多个子系统上的cgroups目录树。
* 子系统（Subsystem）：一个子系统就是一个资源控制器，比如CPU子系统就是控制CPU时间分配的一个控制器。子系统必须附加到一个层级上才能起作用，一个子系统附加到某个层级以后，这个层级上的所有控制族群都受到这个子系统的控制。Cgroup的子系统可以有很多，也在不断增加中。

# Docker 对 cgroups 的使用

## 默认情况

默认情况下，Docker 启动一个容器后，会在 /sys/fs/cgroup 目录下的各个资源目录下生成以容器 ID 为名字的目录（group），比如：

````
/sys/fs/cgroup/cpu/docker/03dd196f415276375f754d51ce29b418b170bd92d88c5e420d6901c32f93dc14
````

此时 cpu.cfs_quota_us 的内容为 -1，表示默认情况下并没有限制容器的 CPU 使用。在容器被 stopped 后，该目录被删除。

运行命令 docker run -d --name web41 --cpu-quota 25000 --cpu-period 100 --cpu-shares 30 training/webapp python app.py 启动一个新的容器，结果：

````
root@devstack:/sys/fs/cgroup/cpu/docker/06bd180cd340f8288c18e8f0e01ade66d066058dd053ef46161eb682ab69ec24# cat cpu.cfs_quota_us
25000
root@devstack:/sys/fs/cgroup/cpu/docker/06bd180cd340f8288c18e8f0e01ade66d066058dd053ef46161eb682ab69ec24# cat tasks
3704
root@devstack:/sys/fs/cgroup/cpu/docker/06bd180cd340f8288c18e8f0e01ade66d066058dd053ef46161eb682ab69ec24# cat cpu.cfs_period_us
2000
````

Docker 会将容器中的进程的 ID 加入到各个资源对应的 tasks 文件中。表示 Docker 也是以上面的机制来使用 cgroups 对容器的 CPU 使用进行限制。

相似地，可以通过 docker run 中 mem 相关的参数对容器的内存使用进行限制：

````
      --cpuset-mems string          MEMs in which to allow execution (0-3, 0,1)
      --kernel-memory string        Kernel memory limit
  -m, --memory string               Memory limit
      --memory-reservation string   Memory soft limit
      --memory-swap string          Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int       Tune container memory swappiness (0 to 100) (default -1)
````

比如  docker run -d --name web42 --blkio-weight 100 --memory 10M --cpu-quota 25000 --cpu-period 2000 --cpu-shares 30 training/webapp python app.py：

````
root@devstack:/sys/fs/cgroup/memory/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410# cat memory.limit_in_bytes
10485760
  root@devstack:/sys/fs/cgroup/blkio/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410# cat blkio.weight
  100
````

目前 docker 已经几乎支持了所有的 cgroups 资源，可以限制容器对包括 network，device，cpu 和 memory 在内的资源的使用，比如：

````
root@devstack:/sys/fs/cgroup# find -iname ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410
./net_prio/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410
./net_cls/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410
./systemd/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410
./hugetlb/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410
./perf_event/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410
./blkio/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410
./freezer/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410
./devices/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410
./memory/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410
./cpuacct/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410
./cpu/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410
./cpuset/docker/ec8d850ebbabaf24df572cb5acd89a6e7a953fe5aa5d3c6a69c4532f92b57410
````

# Docker run 命令中 cgroups 相关命令 

````
block IO:
      --blkio-weight value          Block IO (relative weight), between 10 and 1000
      --blkio-weight-device value   Block IO weight (relative device weight) (default [])
      --cgroup-parent string        Optional parent cgroup for the container
CPU:
      --cpu-percent int             CPU percent (Windows only)
      --cpu-period int              Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int               Limit CPU CFS (Completely Fair Scheduler) quota
  -c, --cpu-shares int              CPU shares (relative weight)
      --cpuset-cpus string          CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string          MEMs in which to allow execution (0-3, 0,1)
Device:    
      --device value                Add a host device to the container (default [])
      --device-read-bps value       Limit read rate (bytes per second) from a device (default [])
      --device-read-iops value      Limit read rate (IO per second) from a device (default [])
      --device-write-bps value      Limit write rate (bytes per second) to a device (default [])
      --device-write-iops value     Limit write rate (IO per second) to a device (default [])
Memory:      
      --kernel-memory string        Kernel memory limit
  -m, --memory string               Memory limit
      --memory-reservation string   Memory soft limit
      --memory-swap string          Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int       Tune container memory swappiness (0 to 100) (default -1)
````

一些说明：

1. cgroup 只能限制 CPU 的使用，而不能保证CPU的使用。也就是说， 使用 cpuset-cpus，可以让容器在指定的CPU或者核上运行，但是不能确保它独占这些CPU；cpu-shares 是个相对值，只有在CPU不够用的时候才其作用。也就是说，当CPU够用的时候，每个容器会分到足够的CPU；不够用的时候，会按照指定的比重在多个容器之间分配CPU。
2. 对内存来说，cgroups 可以限制容器最多使用的内存。使用 -m 参数可以设置最多可以使用的内存。

> to be continue ...


# 参考

http://www.cnblogs.com/sammyliu/p/5875470.html

http://www.cnblogs.com/sammyliu/p/5878973.html

http://www.cnblogs.com/sammyliu/p/5886833.html