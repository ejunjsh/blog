---
title: openstack安装-nova(一)
date: 2017-09-11 19:11:12
tags: [openstack,ubuntu,kvm,nova]
categories: openstack
---
> nova是OpenStack一个核心服务，主要负责虚拟机的各种操作，如启动，销毁，快照，还有选择合适的compute节点部署虚拟机。
> nova中的服务有controller和compute之分，是一对多的关系，即一个controller可以有多个compute，所以一些controller服务要装到`controller`节点上，compute服务可以装到`compute`节点，也可以装到`controller`节点来让它充当一部分compute的能力。
> 由于nova涉及`controller`节点和`compute`节点的安装，所以分了两篇文章来讲解
> 这一篇先介绍怎么安装nova的controller服务在`controller`节点上。

