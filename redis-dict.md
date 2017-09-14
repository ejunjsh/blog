---
title: redis数据结构-字典
date: 2017-09-15 00:59:31
tags: [redis,数据结构,hashtable]
categories: redis
---
> Redis 的字典使用哈希表作为底层实现， 一个哈希表里面可以有多个哈希表节点， 而每个哈希表节点就保存了字典中的一个键值对。

# 哈希表
Redis 字典所使用的哈希表由 dict.h/dictht 结构定义：
````c
typedef struct dictht {

    // 哈希表数组
    dictEntry **table;

    // 哈希表大小
    unsigned long size;

    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;

    // 该哈希表已有节点的数量
    unsigned long used;

} dictht;
````
table 属性是一个数组， 数组中的每个元素都是一个指向 dict.h/dictEntry 结构的指针， 每个 dictEntry 结构保存着一个键值对。
size 属性记录了哈希表的大小， 也即是 table 数组的大小， 而 used 属性则记录了哈希表目前已有节点（键值对）的数量。
sizemask 属性的值总是等于 size - 1 ， 这个属性和哈希值一起决定一个键应该被放到 table 数组的哪个索引上面。
图 4-1 展示了一个大小为 4 的空哈希表 （没有包含任何键值对）。
[![](/images/redis-dict-1.png)](/images/redis-dict-1.png)
<!-- more -->