---
title: TCMalloc：Thread-Caching(线程缓存)Malloc
date: 2018-09-17 22:56:41
tags: [tcmalloc,malloc]
categories: c
---

# 动机

TCMalloc比glibc 2.3 malloc（作为一个名为ptmalloc2的独立库提供）和我测试的其他malloc更快。ptmalloc2在2.8 GHz P4上执行malloc/free操作对（对于小对象）需要大约300纳秒。对于相同的操作对，TCMalloc实现大约需要50纳秒。速度对于malloc实现很重要，因为如果malloc不够快，应用程序编写者倾向于在malloc之上编写自己的自定义空闲列表。这可能导致额外的复杂性和更多的内存使用，除非应用程序编写者非常小心地适当调整空闲列表的大小并从空闲列表中清除空闲对象

TCMalloc还减少了多线程程序的锁争用。对于小型对象，几乎没有争用。对于大型对象，TCMalloc尝试使用细粒度和高效的自旋锁。ptmalloc2还通过使用每线程arena来减少锁争用，但是ptmalloc2使用每线程arena存在很大问题。在ptmalloc2中，内存永远不会从一个arena转移到另一个arena。这可能导致大量浪费的空间。例如，在一个Google应用程序中，第一阶段将为其数据结构分配大约300MB的内存。当第一阶段结束时，第二阶段将在同一地址空间中开始。如果第二阶段被指定为与第一阶段使用的arena不同的arena，此阶段不会重用第一阶段之后剩余的任何内存，并会向地址空间添加另外300MB。在其他应用中也注意到类似的内存爆炸问题。
<!-- more -->
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

# 大对象分配

大对象大小（> 32K）向上舍入到页大小（4K）并由central页堆处理。central页堆也是一个空闲列表数组。对于i < 256，第k个数组元素是一个空闲列表，列表元素由k个页组成。所以第256个数组元素的空闲列表元素是由>= 256个页组成：

[![](http://idiotsky.top/images3/tcmalloc-3.gif)](http://idiotsky.top/images3/tcmalloc-3.gif)

分配一个k个页的内存首先要到第k个空闲列表看看是否满足。如果该空闲列表为空，我们将查看下一个空闲列表，依此类推。最后，如有必要，我们会查看最后一个空闲列表。如果失败，我们从系统中获取内存（使用sbrk，mmap或在/dev/mem映射部分内存）。

如果满足k个页内存分配的是页大小>k，则将分配后剩余部分重新插入页堆中的相应空闲列表中。

# Spans（跨度）

由TCMalloc管理的堆由一组页组成。一系列连续页面由Span对象表示。Span可以分配，也可以是空闲的。如果空闲，则span是页堆链接列表中的条目之一。如果已分配，则它是已传递给应用程序的大对象，或者已分割为一系列小对象的一组页。如果拆分为小对象，则会在Span中记录对象的size-class。

可以使用由页码索引的central数组来查找页面所属的Span。例如，Span a占用2页，Span b占1页，Span c占5页，Span d占3页。

[![](http://idiotsky.top/images3/tcmalloc-4.gif)](http://idiotsky.top/images3/tcmalloc-4.gif)

32位地址空间可以容纳2 ^ 20个4K页面，因此这个中央阵列占用4MB空间，这似乎是可以接受的。在64位计算机上，我们使用3层基数树(radix tree)而不是数组来从页码映射到相应的Span指针。

# 释放

当一个对象被释放时，我们计算其页码并在central数组中查找它以找到相应的span对象。span告诉我们对象是否很小，如果对象很小，span告诉我们对象的size-class。如果对象很小，我们将它插入当前线程的线程缓存中的相应空闲列表中。如果线程缓存现在超过预定大小（默认为2MB），我们运行垃圾收集器，将未使用的对象从线程缓存移动到central空闲列表中。

如果对象很大，则span告诉我们对象覆盖的页范围。假设这个范围是[p,q]。我们还为span查找页p-1和页q+1。如果这些相邻span中的任何一个是空闲的，我们将它们与[p,q]的span合并 。生成的span将插入页堆中的相应空闲列表中。

# 小对象的central空闲列表

如前所述，我们为每个size-class保留一个central空闲列表。每个central空闲列表都被组织为一个两级数据结构：一组span，以及每个span的空闲对象的链表。

从central空闲列表分配对象,是通过从某个span的链表中删除第一个条目。（如果所有span都有空链表，则首先从central页堆分配适当大小的span。）

将对象返回到central空闲列表，是通过将对象添加到其包含span的链表。如果链表长度现在等于span中的小对象总数，则此span现在完全空闲并返回到页堆。

# 线程缓存的垃圾回收

当缓存中所有对象的组合大小超过2MB时，将对线程缓存进行垃圾收集。随着线程数量的增加，垃圾收集阈值会自动降低，这样我们就不会在具有大量线程的程序中浪费过多的内存。

我们遍历缓存中的所有空闲列表，并将一些对象从空闲列表移动到相应的central列表。

从空闲列表移动的对象数量是使用每个列表的低水位标记L决定的。 L记录自上次垃圾回收以来列表的最小长度。请注意，我们可以通过L最后一个垃圾收集中的对象缩短列表，而无需对central表进行任何额外访问。我们使用此历史记录作为将来访问的预测器，并将L/2对象从线程缓存空闲列表移动到相应的central空闲列表。该算法具有良好的属性，如果线程停止使用特定大小，则该大小的所有对象将快速从线程高速缓存移动到central空闲列表，其他线程可以使用它们。

# 参考

翻译 http://goog-perftools.sourceforge.net/doc/tcmalloc.html
