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

> to be continue ...

# 参考

http://www.cnblogs.com/sammyliu/p/5875470.html

