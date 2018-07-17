---
title: jvm full gc 解惑
date: 2017-11-06 18:03:15
tags: [jvm,java,gc]
categories: java
---
> 一直以来，都觉得full gc就是对old区的gc，然后就打脸了。。。。

下面先引用R大的原文：
>针对HotSpot VM的实现，它里面的GC其实准确分类只有两大种：
>* Partial GC：并不收集整个GC堆的模式
>    * Young GC：只收集young gen的GC
>    * Old GC：只收集old gen的GC。只有CMS的concurrent collection是这个模式
>    * Mixed GC：收集整个young gen以及部分old gen的GC。只有G1有这个模式
>* Full GC：收集整个堆，包括young gen、old gen、perm gen（如果存在的话）等所有部分的模式。

<!-- more -->
>Major GC通常是跟full GC是等价的，收集整个GC堆。但因为HotSpot VM发展了这么多年，外界对各种名词的解读已经完全混乱了，当有人说“major GC”的时候一定要问清楚他想要指的是上面的full GC还是old GC。

>最简单的分代式GC策略，按HotSpot VM的serial GC的实现来看，触发条件是：
>* young GC：当young gen中的eden区分配满的时候触发。注意young GC中有部分存活对象会晋升到old gen，所以young GC后old gen的占用量通常会有所升高。
>* full GC：当准备要触发一次young GC时，如果发现统计数据说之前young GC的平均晋升大小比目前old gen剩余的空间大，则不会触发young GC而是转为触发full GC（因为HotSpot VM的GC里，除了CMS的concurrent collection之外，其它能收集old gen的GC都会同时收集整个GC堆，包括young gen，所以不需要事先触发一次单独的young GC）；或者，如果有perm gen的话，要在perm gen分配空间但已经没有足够空间时，也要触发一次full GC；或者System.gc()、heap dump带GC，默认也是触发full GC。

>HotSpot VM里其它非并发GC的触发条件复杂一些，不过大致的原理与上面说的其实一样。当然也总有例外。Parallel Scavenge（-XX:+UseParallelGC）框架下，默认是在要触发full GC前先执行一次young GC，并且两次GC之间能让应用程序稍微运行一小下，以期降低full GC的暂停时间（因为young GC会尽量清理了young gen的死对象，减少了full GC的工作量）。控制这个行为的VM参数是-XX:+ScavengeBeforeFullGC。这是HotSpot VM里的奇葩嗯。

>并发GC的触发条件就不太一样。以CMS GC为例，它主要是定时去检查old gen的使用量，当使用量超过了触发比例就会启动一次CMS GC，对old gen做并发收集。

总结一下：
1. full gc 是全区回收(打脸)
2. full gc 之前不会触发young gc（一般情况）

再引用下别的大牛的话：
>先看一下HotSpot VM的GC家族的组合示意图：
>[![](http://idiotsky.top/images1/java-fullgc-1.jpg)](http://idiotsky.top/images1/java-fullgc-1.jpg)
>不同的GC组合套装之中，具备Full GC能力的大概有三种：
>1. ParallelOld(PSMarkSweep)
>2. Serial Old(MarkSweep)
>3. "?"所代表的G1

>暂不考虑G1，除了CMS具备在年老代进行Major GC之外，其他情况下年老代的GC都是由Full GC触发的。Full GC的收集范围包含整个Heap区域( Eden + S1 + S2 + Tenured)，它发生时Mutator停止工作——Stop The World。对于Serial Collector，它采用MSC(Mark-Sweep-Compact)的算法对全堆进行Full GC，在HotSpot VM的实现中，主要用MarkSweep这个类来实现；对于Parallel Collector而言，PSMarkSweep是多线程的MarkSweep，名不副实，这玩意儿其实是个实现了Lisp2的Mark-Compact GC算法。PSMarkSweep有个特殊的地方是如果配置了__ScavengeBeforeFullGC__ 这个flag，则会在Full GC之前对年轻代进行一次Minor GC；其他情况根本不需要Full GC之前先执行Minor GC，Full GC会对年轻代发起GC。Full GC前后Heap的对比示意参见：
>[![](http://idiotsky.top/images1/java-fullgc-2.jpg)](http://idiotsky.top/images1/java-fullgc-2.jpg)

>可见，一般情况下，年轻代的存活对象都被Compact到了年老代，所以，你看到年轻代都被清空了；只有当年老代满了的时候，才会Compact到Eden区域。

>对于Concurrent Collector，CMS在Remark Phase，可以通过设置 __CMSScavengeBeforeRemark__ 在remark之前先行YGC，这给了CMS在Major GC时触发Minor GC的机会，但这个Flag默认是false；当CMS发生 __Concurrent Mode Failure__ 时，CMS会退化为Serial Old GC，从而采用与Serial Collector相同的算法进行Full GC。CMS发生 __Concurrent Mode Failure__ 的原因：1. 因为是并发收集，所以Mutator仍可能在不断占用年老代的空间，当然还包括这一趟无法收集的Float Garbage会占用内存空间，如果年老代空间被占满但并发收集还未结束，就会发生并发模式失败；2. 因为CMS采用的是Mark Sweep算法，本身内存碎片化无法解决，很可能发生大对象分配时没有连续空间，或者本身剩余空间不够大对象分配时，也会发生并发模式失败。


再总结下:
1. Serial Old 做full gc 时候不会执行young gc，而 ParallelOld 会根据 __ScavengeBeforeFullGC__ 来决定是否在full gc前执行一次young gc
2. CMS 有自己的major gc，单独执行old区的gc，但是如果 __Concurrent Mode Failure__ 的话，就还是老老实实做Serial Old的full gc吧。
3. CMS 的 __CMSScavengeBeforeRemark__ 标记决定了是否在remark阶段之前执行一次young gc（网上说这个标记还能解决跨代引用问题），

参考 https://www.zhihu.com/question/62604570
参考 https://www.zhihu.com/question/41922036

