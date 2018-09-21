---
title: TCMalloc：Thread-Caching Malloc
date: 2018-09-17 22:56:41
tags: [tcmalloc]
categories: c
---

# 动机

TCMalloc比glibc 2.3 malloc（作为一个名为ptmalloc2的独立库提供）和我测试的其他malloc更快。ptmalloc2在2.8 GHz P4上执行malloc/free操作对（对于小对象）需要大约300纳秒。对于相同的操作对，TCMalloc实现大约需要50纳秒。速度对于malloc实现很重要，因为如果malloc不够快，应用程序编写者倾向于在malloc之上编写自己的自定义空闲列表。这可能导致额外的复杂性和更多的内存使用，除非应用程序编写者非常小心地适当调整空闲列表的大小并从空闲列表中清除空闲对象

TCMalloc还减少了多线程程序的锁争用。对于小型对象，几乎没有争用。对于大型对象，TCMalloc尝试使用细粒度和高效的自旋锁。ptmalloc2还通过使用每线程arena来减少锁争用，但是ptmalloc2使用每线程arena存在很大问题。在ptmalloc2中，内存永远不会从一个arena转移到另一个arena。这可能导致大量浪费的空间。例如，在一个Google应用程序中，第一阶段将为其数据结构分配大约300MB的内存。当第一阶段结束时，第二阶段将在同一地址空间中开始。如果第二阶段被指定为与第一阶段使用的arena不同的arena，此阶段不会重用第一阶段之后剩余的任何内存，并会向地址空间添加另外300MB。在其他应用中也注意到类似的内存爆炸问题。

TCMalloc的另一个好处是小对象都能节省更多空间。例如，分配N个8字节对象，才使用大约8N * 1.01字节的空间。即，一个百分之一的空间开销。而ptmalloc2为每个对象使用一个四字节的头，并且（我认为）将大小四舍五入为8个字节的倍数，最后使用16N字节。

# 用法
要使用TCmalloc，只需通过“-ltcmalloc”链接器标志将tcmalloc链接到您的应用程序。

您可以使用LD_PRELOAD在自己没有编译的应用程序中使用tcmalloc：
````
   $ LD_PRELOAD =“/ usr / lib / libtcmalloc.so
````

LD_PRELOAD很棘手，我们不一定推荐这种使用模式。

TCMalloc还包括[检查器](http://goog-perftools.sourceforge.net/doc/heap_checker.html) 和[堆分析器](http://goog-perftools.sourceforge.net/doc/heap_profiler.html)。

如果您更喜欢链接不包含堆分析器和检查器的TCMalloc版本（可能减少静态二进制文件的二进制大小），则可以链接libtcmalloc_minimal 。

# 概观

TCMalloc为每个线程分配线程本地缓存。线程本地缓存满足小内存的分配。根据需要将对象从central数据结构移动到线程本地缓存中，并使用周期性垃圾收集将内存从线程本地缓存迁移回central数据结构。

[![](http://idiotsky.top/images3/tcmalloc-1.png)](http://idiotsky.top/images3/tcmalloc-1.png)

> to be continue

翻译 http://goog-perftools.sourceforge.net/doc/tcmalloc.html
