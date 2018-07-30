---
title: java-concurrenthashmap
date: 2018-07-28 15:31:20
tags: [conccurenthashmap,数据结构,java]
categories: java
---
> 只剩下这个没有mark了，偏偏这时候，面试被问到，悲剧了😢

<!-- more -->

# 背景

由于`hashmap`不是线程安全的，而`hashtable`就是加锁粒度很大，所以jdk7就衍生了这个`concurrenthashmap`。

而它在jdk7中的实现和jdk8的实现完全不同，所以接下来分jdk7和jdk8的实现来说说

# jdk7的conccurenthashmap

整个 ConcurrentHashMap 由一个个 Segment 组成，Segment 代表”部分“或”一段“的意思，所以很多地方都会将其描述为分段锁。注意，行文中，我很多地方用了“槽”来代表一个 segment。

简单理解就是，ConcurrentHashMap 是一个 Segment 数组，Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。

## 

# jdk8的conccurenthashmap
