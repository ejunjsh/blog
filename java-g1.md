---
title: java垃圾收集器G1入门
date: 2018-07-28 18:54:54
tags: [G1,java]
categories: java
---

# G1垃圾收集器

Garbage-First（G1）收集器是一种服务器式垃圾收集器，适用于具有大容量存储器的多处理器机器。它以高概率满足垃圾收集（GC）暂停时间目标，同时实现高吞吐量。Oracle JDK 7 Update 4及更高版本完全支持G1垃圾收集器。G1收集器专为以下应用而设计：

* 可以与CMS收集器等应用程序线程同时运行。
* 紧凑的自由空间，没有长时间的GC引起的暂停时间
* 需要更可预测的GC暂停持续时间。
* 不想牺牲很多吞吐量性能。
* 不需要更大的Java堆。

G1计划作为Concurrent Mark-Sweep Collector（CMS）的长期替代品。将G1与CMS进行比较，存在差异，使G1成为更好的解决方案。一个区别是G1是压缩收集器。G1足够紧凑以完全避免使用细粒度的自由列表进行分配，而是依赖于区域。这大大简化了收集器的部分，并且主要消除了潜在的碎片问题。此外，G1提供比CMS收集器更可预测的垃圾收集暂停，并允许用户指定所需的暂停目标。

## G1操作概览

旧的垃圾收集器（串行，并行，CMS）都将堆构建为三个部分：年轻代，旧代和永久生成固定内存大小。

[![](http://idiotsky.top/images3/G1-1.png)](http://idiotsky.top/images3/G1-1.png)

<!-- more -->

所有内存对象都在这三个部分之一里结束。

G1收集器采用不同的方法。

[![](http://idiotsky.top/images3/G1-2.png)](http://idiotsky.top/images3/G1-2.png)

堆被分区为一组大小相等的堆区域，每个区域都是一个连续的虚拟内存区域。某些区域集被赋予与旧收集器中相同的角色（eden，survivor，old），但它们没有固定的大小。这为内存使用提供了更大的灵活性。

执行垃圾收集时，G1以类似于CMS收集器的方式运行。G1执行并发全局标记阶段以确定整个堆中对象的活跃度。在标记阶段完成之后，G1知道哪些区域基本上是空的。它首先收集在这些区域，这通常会产生大量的自由空间。这就是为什么这种垃圾收集方法称为Garbage-First。顾名思义，G1将其收集和压缩活动集中在堆的可能充满可回收对象的区域，即垃圾。G1使用暂停预测模型来满足用户定义的暂停时间目标，并根据指定的暂停时间目标选择要收集的区域数。

由G1确定为回收成熟的区域是使用疏散收集的垃圾。G1将对象从堆的一个或多个区域复制到堆上的单个区域，并且在此过程中压缩并释放内存。这种疏散在多处理器上并行执行，以减少暂停时间并提高吞吐量。因此，对于每次垃圾收集，G1会持续工作以减少碎片，在用户定义的暂停时间内工作。这超出了以前两种方法的能力。CMS（Concurrent Mark Sweep）垃圾收集器不进行压缩。ParallelOld垃圾收集仅执行整堆压缩，这会导致相当长的暂停时间。

值得注意的是，G1不是实时收集器。它以高概率但不是绝对确定性满足设定的暂停时间目标。基于先前集合的数据，G1估计可以在用户指定的目标时间内收集多少个区域。因此，收集器具有收集区域的成本的相当准确的模型，并且它使用该模型来确定在停留在暂停时间目标内时要收集哪些区域和多少区域。

注意： G1具有并发（与应用程序线程一起运行，例如，细化，标记，清理）和并行（多线程，例如，停止世界）阶段。完整的垃圾收集仍然是单线程的，但如果调整得当，您的应用程序应该避免使用完整的GC。

## G1足迹

如果从ParallelOldGC或CMS收集器迁移到G1，您可能会看到更大的JVM进程大小。这主要与“记帐”数据结构有关，例如记忆集和集合集。

记住集或RSet跟踪对象引用到给定区域。堆中每个区域有一个RSet。RSet支持并行和独立收集区域。RSets的总体足迹影响小于5％。

Collection设置或CSets将在GC中收集的区域集。在GC期间，CSet中的所有实时数据都被撤离（复制/移动）。区域集可以是伊甸园，幸存者和/或老一代。CSets对JVM的大小影响不到1％。

## 推荐的G1用例

G1的第一个重点是为运行需要具有有限GC延迟的大堆的应用程序的用户提供解决方案。这意味着堆大小约为6GB或更大，稳定且可预测的暂停时间低于0.5秒。

如果应用程序具有以下一个或多个特征，那么今天使用CMS或ParallelOldGC垃圾收集器运行的应用程序将有利于切换到G1。

* 完整的GC持续时间太长或太频繁。
* 对象分配率或促销率差异很大。
* 不期望的长垃圾收集或压实暂停（超过0.5到1秒）

注意：如果您使用的是CMS或ParallelOldGC，并且您的应用程序没有经历长时间的垃圾收集暂停，那么与您当前的收集器保持联系是可以的。更改为G1收集器不是使用最新JDK的必要条件。

# 回顾CMS

并发标记扫描（CMS）收集器（也称为并发低暂停收集器）收集终生代。它尝试通过与应用程序线程同时执行大多数垃圾收集工作来最小化由于垃圾收集而导致的暂停。通常，并发低暂停收集器不会复制或压缩活动对象。无需移动活动对象即可完成垃圾收集。如果碎片成为问题，请分配更大的堆。

注意：年轻代的CMS收集器使用与并行收集器相同的算法。

## CMS收集阶段

CMS收集器在旧一代堆上执行以下阶段：


相	|描述
---|-----
（1）初始标记（停止世界事件）|	旧一代中的对象被“标记”为可到达的，包括可以从年轻一代到达的那些对象。相对于较小的收集暂停时间，暂停时间通常较短。
（2）并发标记|	在Java应用程序线程执行时，同时遍历可达对象的终身生成对象图。从标记的对象开始扫描，并传递标记从根可到达的所有对象。变换器在并发阶段2,3和5期间执行，并且在这些阶段（包括提升的对象）中在CMS生成中分配的任何对象立即被标记为实时。
（3）备注（停止世界事件）|	查找并发标记阶段遗漏的对象，因为在并发收集器完成对该对象的跟踪之后，Java应用程序线程更新到对象。
（4）并发扫描|	收集在标记阶段标识为无法访问的对象。死对象的集合将对象的空间添加到空闲列表以供稍后分配。此时可能会发生死物的合并。请注意，不会移动活动对象。
（5）重置|	通过清除数据结构准备下一个并发收集。

## 回顾垃圾收集步骤

接下来，让我们一步一步查看CMS收集器操作。

### CMS收集器的堆结构

堆被分成三个空间。

[![](http://idiotsky.top/images3/G1-3.png)](http://idiotsky.top/images3/G1-3.png)

年轻一代分为伊甸园和两个幸存者空间。老一代是一个连续的空间。对象集合就地完成了。除非有完整的GC，否则不会进行压缩。

### Young GC如何在CMS中运行

年轻一代是浅绿色，老一代是蓝色。如果您的应用程序已运行一段时间，这就是CMS的样子。物体散落在老一代区域周围。

[![](http://idiotsky.top/images3/G1-4.png)](http://idiotsky.top/images3/G1-4.png)

使用CMS，旧的对象被释放到位。他们不会四处走动。除非有完整的GC，否则不会压缩空间。

### 年轻一代集合

活体对象从伊甸园空间和幸存者空间复制到另一个幸存者空间。任何已达到其老化阈值的旧对象都将升级为旧一代。

[![](http://idiotsky.top/images3/G1-5.png)](http://idiotsky.top/images3/G1-5.png)

### 年轻的GC之后

在年轻的GC之后，伊甸园空间被清除，其中一个幸存者空间被清除。

[![](http://idiotsky.top/images3/G1-6.png)](http://idiotsky.top/images3/G1-6.png)

新推荐的对象在图上以深蓝色显示。绿色物体是幸存的尚未升级为老一代的年轻一代物体。

### 与CMS的老一代集合

两次停止世界事件发生：初始标记和备注。当老一代达到一定的入住率时，CMS就会被启动。

[![](http://idiotsky.top/images3/G1-7.png)](http://idiotsky.top/images3/G1-7.png)

（1）初始标记是短暂停顿阶段，其中标记了实时（可到达）对象。（2）并发标记在应用程序继续执行时查找活动对象。最后，在（3）备注阶段，发现在前一阶段（2）并发标记期间遗漏的对象。

### 老一代收藏 - 并发扫描

在前一阶段未标记的对象将被释放到位。没有压缩。

[![](http://idiotsky.top/images3/G1-8.png)](http://idiotsky.top/images3/G1-8.png)

注意：未标记的对象==死对象

### 老一代收藏 - 清扫后

在（4）清扫阶段之后，您可以看到已经释放了大量内存。您还会注意到没有进行压缩。

[![](http://idiotsky.top/images3/G1-9.png)](http://idiotsky.top/images3/G1-9.png)

最后，CMS收集器将在（5）重置阶段进行，并等待下一次达到GC阈值。

# G1收集步骤

G1收集器采用不同的方法来分配堆。下面的图片逐步回顾G1系统。

## G1堆结构

堆是一个分成许多固定大小区域的内存区域。

[![](http://idiotsky.top/images3/G1-10.png)](http://idiotsky.top/images3/G1-10.png)

区域大小由JVM在启动时选择。JVM通常针对大约2000个区域，大小从1到32Mb不等。

## G1堆分配

实际上，这些区域被映射为伊甸园，幸存者和老一代空间的逻辑表示。

[![](http://idiotsky.top/images3/G1-11.png)](http://idiotsky.top/images3/G1-11.png)

图中的颜色显示哪个区域与哪个角色相关联。活动对象从一个区域撤离（即，复制或移动）到另一个区域。区域被设计为在停止所有其他应用程序线程的情况下并行收集。

如图所示，区域可以分配到伊甸园，幸存者和老一代地区。此外，还有第四种被称为Humongous区域的物体。这些区域设计用于容纳标准区域大小50％或更大的对象。它们存储为一组连续的区域。最后，最后一种类型的区域将是堆的未使用区域。

注意：在撰写本文时，尚未优化收集大量物体。因此，您应该避免创建此大小的对象。

## G1中的年轻一代

堆被分成大约2000个区域。最小大小为1Mb，最大大小为32Mb。蓝色区域包含老一代物体，绿色区域包含年轻一代物体。

[![](http://idiotsky.top/images3/G1-12.png)](http://idiotsky.top/images3/G1-12.png)

请注意，区域不需要像旧垃圾收集器那样连续。

## G1中的年轻GC

将活动对象撤离（即，复制或移动）到一个或多个幸存者区域。如果满足老化阈值，则将一些对象提升到旧生成区域。

[![](http://idiotsky.top/images3/G1-13.png)](http://idiotsky.top/images3/G1-13.png)

这是一个停止世界（STW）暂停。计算下一个年轻GC的伊甸园大小和幸存者大小。保留会计信息以帮助计算大小。像暂停时间目标这样的事情被考虑在内。

这种方法可以很容易地调整区域大小，使它们根据需要变大或变小。

## 与G1结束的年轻GC

活体物体已被疏散到幸存者地区或老一代地区。

[![](http://idiotsky.top/images3/G1-14.png)](http://idiotsky.top/images3/G1-14.png)

最近推出的对象以深蓝色显示。绿色的幸存者地区。

总之，关于G1中的年轻一代可以说如下：

* 堆是分成区域的单个内存空间。
* 年轻代存储器由一组非连续区域组成。这样可以在需要时轻松调整大小。
* 年轻一代的垃圾收集品，或年轻的GC，都是阻止世界事件。为操作停止所有应用程序线程。
* 年轻的GC使用多个线程并行完成。
* 活动对象被复制到新的幸存者或旧生成区域。

## 与G1的老一代集合

与CMS收集器一样，G1收集器设计为老一代对象的低暂停收集器。下表描述了旧一代的G1收集阶段。

## G1收集阶段 - 并行标记循环阶段

G1收集器在旧一代堆上执行以下阶段。请注意，某些阶段是年轻一代集合的一部分。


相	|描述
----|-----
（1）初始标记（停止世界事件）|	这是世界事件的一个停止。使用G1，它可以搭载在正常的年轻GC上。标记可能引用旧一代对象的幸存者区域（根区域）。
（2）根区扫描	|扫描幸存者区域以引用旧代。应用程序继续运行时会发生这种情况。必须在年轻GC发生之前完成该阶段。
（3）并发标记|	在整个堆上查找实时对象。这在应用程序运行时发生。年代垃圾收集可以中断此阶段。
（4）备注（停止世界事件）|	完成堆中活动对象的标记。使用称为开始时快照（SATB）的算法，该算法比CMS收集器中使用的算法快得多。
（5）清理（停止世界事件和并发）	|对活动对象和完全自由区域执行会计。（阻止世界）擦洗记忆集。（阻止世界）重置空白区域并将其返回到空闲列表。（同时）
（*）复制（停止世界事件）|	这些是世界暂停撤离或将活动对象复制到新的未使用区域的停止。这可以通过记录为的年轻代区域来完成[GC pause (young)]。或者记录为的年轻和老一代地区[GC Pause (mixed)]。

## G1老年代收集步骤

通过定义阶段，让我们看看他们如何与G1收集器中的老一代进行交互。

### 初始标记阶段

活动对象的初始标记背负在年轻一代垃圾收集上。在日志中，这被标记为GC pause (young)(inital-mark)。

[![](http://idiotsky.top/images3/G1-15.png)](http://idiotsky.top/images3/G1-15.png)

### 并行标记阶段

如果找到空区域（由“X”表示），则在备注阶段立即将它们移除。此外，计算确定活跃度的“会计”信息。

[![](http://idiotsky.top/images3/G1-16.png)](http://idiotsky.top/images3/G1-16.png)

### 备注阶段

空区域被移除并回收。现在计算所有地区的区域活跃度。

[![](http://idiotsky.top/images3/G1-17.png)](http://idiotsky.top/images3/G1-17.png)


### 复制/清理阶段

G1选择具有最低“活跃度”的区域，那些可以最快收集的区域。然后，这些区域与年轻的GC同时收集。这在日志中表示为[GC pause (mixed)]。因此，同时收集年轻一代和老一代。

[![](http://idiotsky.top/images3/G1-18.png)](http://idiotsky.top/images3/G1-18.png)

### 复制/清理阶段后

选择的区域已经被收集并压缩成图中所示的深蓝色区域和深绿色区域。

[![](http://idiotsky.top/images3/G1-19.png)](http://idiotsky.top/images3/G1-19.png)

## 老一代GC概述
总之，我们可以对老一代的G1垃圾收集提出几点要点。

* 并行标记阶段
    * 在应用程序运行时同时计算活动信息。
    * 该活跃度信息识别在疏散暂停期间最佳回收的区域。
    * 没有类似于CMS的席卷阶段。
* 备注阶段
    * 使用开始时快照（SATB）算法，该算法比CMS使用的算法快得多。
    * 完全空的区域被回收。
* 复制/清理阶段
    * 年轻一代和老一代同时被收回。
    * 老一代地区的选择取决于他们的生活。
 
# 命令行选项和最佳实践

在本节中，我们来看看G1的各种命令行选项。

## 基本命令行
要启用G1 Collector，请使用： -XX:+UseG1GC

下面是一个示例命令行，用于启动JDK演示和示例下载中包含的Java2Demo：
java -Xmx50m -Xms50m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -jar c:\javademos\demo\jfc\Java2D\Java2demo.jar

## 键命令行开关
-XX:+UseG1GC - 告诉JVM使用G1垃圾收集器。

-XX：MaxGCPauseMillis = 200 - 设置最大GC暂停时间的目标。这是一个软目标，JVM将尽最大努力实现它。因此，有时不会达到暂停时间目标。默认值为200毫秒。

-XX：InitiatingHeapOccupancyPercent = 45 - 启动并发GC循环的（整个）堆占用百分比。G1使用它来根据整个堆的占用情况触发并发GC循环，而不仅仅是其中一代。值0表示“执行常数GC循环”。默认值为45（即45％已满或已占用）。

## 最佳实践
使用G1时，您应该遵循一些最佳实践。

不要设置年轻一代的大小

-Xmn使用G1收集器的默认行为通过meddles 显式设置年轻代大小。

G1将不再遵守集合的暂停时间目标。因此，实质上，设置年轻代大小会禁用暂停时间目标。
G1不再能够根据需要扩展和收缩年轻一代的空间。由于尺寸是固定的，因此不能对尺寸进行任何改变。
响应时间指标

不要使用平均响应时间（ART）作为设置的度量XX:MaxGCPauseMillis=<N>，而应考虑设置值，该值将在90％的时间内达到目标或更高。这意味着90％的用户提出请求的响应时间不会高于目标。请记住，暂停时间是一个目标，并不能保证始终满足。

什么是疏散失败？

在GC期间JVM在堆区域中用完幸存者和提升对象时发生的升级失败。堆无法扩展，因为它已经处于最大值。这是使用时，在GC日志中所示-XX:+PrintGCDetails的to-space overflow。这很贵！

GC仍然需要继续，因此必须释放空间。
未成功复制的对象必须在适当的位置使用。
必须重新生成对CSet中的区域的RSets的任何更新。
所有这些步骤都很昂贵。
如何避免疏散失败

为避免疏散失败，请考虑以下选项。

* 增加堆大小
    * 增加-XX:G1ReservePercent=n，默认为10。
    * G1通过尝试保留备用存储器来创建假天花板，以防需要更多“空间”。
* 提前开始标记周期
* 使用该-XX:ConcGCThreads=n选项增加标记线程的数量。

## G1 GC开关的完整列表

这是G1 GC开关的完整列表。请记住使用上面列出的最佳做法。

选项和默认值|	描述
----|-----
-XX：+ UseG1GC|	使用垃圾优先（G1）收集器
-XX:MaxGCPauseMillis=n|设置最大GC暂停时间的目标。这是一个软目标，JVM将尽最大努力实现它。
-XX：InitiatingHeapOccupancyPercent =n	|启动并发GC循环的（整个）堆占用百分比。它由GC使用，它基于整个堆的占用而不仅仅是其中一代（例如，G1）触发并发GC循环。值0表示“执行常数GC循环”。默认值为45。
-XX：NewRatio =n	|新旧一代的比例。默认值为2。
-XX：SurvivorRatio =n	|伊甸园/幸存者空间大小的比率。默认值为8。
-XX：MaxTenuringThreshold =n	|终身临界值的最大值。默认值为15。
-XX：ParallelGCThreads =n|	设置垃圾收集器并行阶段使用的线程数。默认值因运行JVM的平台而异。
-XX：ConcGCThreads =n|	并发垃圾收集器将使用的线程数。默认值因运行JVM的平台而异。
-XX：G1ReservePercent =n	|设置保留为false上限的堆的数量，以减少促销失败的可能性。默认值为10。
-XX：G1HeapRegionSize =n	|使用G1，Java堆被细分为大小均匀的区域。这设置了各个子部门的大小。根据堆大小，符合人体工程学地确定此参数的默认值。最小值为1Mb，最大值为32Mb。


部分翻译 http://www.oracle.com/technetwork/tutorials/tutorials-1876574.html