---
title: linux epoll原理
date: 2017-09-11 01:35:15
tags: [linux,nio,epoll]
categories: linux
---
> epoll是Linux下的一种IO多路复用技术，可以非常高效的处理数以百万计的socket句柄。 

先看看使用c封装的3个epoll系统调用：

* __int epoll_create(int size)__
    epoll\_create建立一个epoll对象。参数size是内核保证能够正确处理的最大句柄数，多于这个最大数时内核可不保证效果。
* __int epoll\_ctl(int epfd, int op, int fd, struct epoll_event *event)__
    epoll\_ctl可以操作epoll_create创建的epoll，如将socket句柄加入到epoll中让其监控，或把epoll正在监控的某个socket句柄移出epoll。
* __int epoll_wait(int epfd, struct epoll_event *events,int maxevents, int timeout)__
    epoll\_wait在调用时，在给定的timeout时间内，所监控的句柄中有事件发生时，就返回用户态的进程。

<!-- more -->

大概看看epoll内部是怎么实现的：
1. epoll初始化时，会向内核注册一个文件系统，用于存储被监控的句柄文件，调用epoll_create时，会在这个文件系统中创建一个file节点。同时epoll会开辟自己的内核高速缓存区，以红黑树的结构保存句柄，以支持快速的查找、插入、删除。还会再建立一个list链表，用于存储准备就绪的事件。
2. 当执行epoll_ctl时，除了把socket句柄放到epoll文件系统里file对象对应的红黑树上之外，还会给内核中断处理程序注册一个回调函数，告诉内核，如果这个句柄的中断到了，就把它放到准备就绪list链表里。所以，当一个socket上有数据到了，内核在把网卡上的数据copy到内核中后，就把socket插入到就绪链表里。
3. 当epoll_wait调用时，仅仅观察就绪链表里有没有数据，如果有数据就返回，否则就sleep，超时时立刻返回。

epoll的两种工作模式：
* __LT__：level-trigger，水平触发模式，只要某个socket处于readable/writable状态，无论什么时候进行epoll_wait都会返回该socket。
* __ET__：edge-trigger，边缘触发模式，只有某个socket从unreadable变为readable或从unwritable变为writable时，epoll_wait才会返回该socket。

socket读数据
[![](http://idiotsky.top/images/epoll-mechanism-1.png)](http://idiotsky.top/images/epoll-mechanism-1.png)

socket写数据
[![](http://idiotsky.top/images/epoll-mechanism-2.png)](http://idiotsky.top/images/epoll-mechanism-2.png)

参考 http://www.jianshu.com/p/0d497fe5484a