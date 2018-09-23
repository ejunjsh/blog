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
<!-- more -->
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

[![](http://idiotsky.top/images3/tcmalloc-1.gif)](http://idiotsky.top/images3/tcmalloc-1.gif)

TCMalloc以不同于较大对象的方式处理大小<= 32K（“小”对象）的对象。使用页级分配器直接从central堆分配大对象（页是4K对齐的内存区域）。即，大对象总是页对齐并占据整数页。

可以将一系列页划分为一系列小对象，每个小对象的大小相同。例如，一页（4K）可以被分成32个大小为128字节的对象。

# 小对象分配

每个小对象大小映射到大约170个可分配size-class中的一个。例如，961到1024字节范围内的所有分配都向上舍入到1024.size-class间隔开，以便小的大小分隔8个字节，较大的大小分隔16个字节，甚至更大的大小分隔32个字节，依此类推。最大间距（对于大小> = ~2K）是256个字节。
线程缓存包含每个size-class的单​​独的空闲对象链表。

[![](http://idiotsky.top/images3/tcmalloc-2.gif)](http://idiotsky.top/images3/tcmalloc-2.gif)

分配小对象时：（1）我们将其大小映射到相应的size-class。（2）查找当前线程的线程缓存中的相应空闲列表。（3）如果空闲列表不为空，我们从列表中删除第一个对象并将其返回。遵循此快速路径时，TCMalloc根本不会获取任何锁。这有助于显着加速分配，因为加锁/解锁对在2.8 GHz Xeon上大约需要100纳秒。

如果空闲列表为空：（1）我们从这个size-class的central空闲列表中获取一堆对象（所有线程共享central空闲列表）。（2）将它们放在线程本地空闲列表中。（3）将一个新获取的对象返回给应用程序。

如果central空闲列表也是空的：（1）我们从central页分配器分配一系列页。（2）将一系列页分成这个size-class的一组对象。（3）将新对象放在central空闲列表中。（4）和以前一样，将其中一些对象移动到线程本地空闲列表中。

> to be continue

翻译 http://goog-perftools.sourceforge.net/doc/tcmalloc.html
