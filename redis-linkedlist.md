---
title: redis数据结构-链表
date: 2017-09-14 23:59:31
tags: [redis,数据结构,链表]
categories: redis
---
> 列表键的底层就是一个链表

# 链表节点
每个链表节点使用一个 adlist.h/listNode 结构来表示：
````c
typedef struct listNode {

    // 前置节点
    struct listNode *prev;

    // 后置节点
    struct listNode *next;

    // 节点的值
    void *value;

} listNode;
````
多个 listNode 可以通过 prev 和 next 指针组成双端链表， 如图 3-1 所示。
[![](http://idiotsky.me/images/redis-linkedlist-1.png)](http://idiotsky.me/images/redis-linkedlist-1.png)
<!-- more -->
# 链表
虽然仅仅使用多个 listNode 结构就可以组成链表， 但使用 adlist.h/list 来持有链表的话， 操作起来会更方便：
````c
typedef struct list {

    // 表头节点
    listNode *head;

    // 表尾节点
    listNode *tail;

    // 链表所包含的节点数量
    unsigned long len;

    // 节点值复制函数
    void *(*dup)(void *ptr);

    // 节点值释放函数
    void (*free)(void *ptr);

    // 节点值对比函数
    int (*match)(void *ptr, void *key);

} list;
````
list 结构为链表提供了表头指针 head 、表尾指针 tail ， 以及链表长度计数器 len ， 而 dup 、 free 和 match 成员则是用于实现多态链表所需的类型特定函数：
* dup 函数用于复制链表节点所保存的值；
* free 函数用于释放链表节点所保存的值；
* match 函数则用于对比链表节点所保存的值和另一个输入值是否相等。

图 3-2 是由一个 list 结构和三个 listNode 结构组成的链表：
[![](http://idiotsky.me/images/redis-linkedlist-2.png)](http://idiotsky.me/images/redis-linkedlist-2.png)

# 总结
Redis 的链表实现的特性可以总结如下：
* 双端： 链表节点带有 prev 和 next 指针， 获取某个节点的前置节点和后置节点的复杂度都是 O(1) 。
* 无环： 表头节点的 prev 指针和表尾节点的 next 指针都指向 NULL ， 对链表的访问以 NULL 为终点。
* 带表头指针和表尾指针： 通过 list 结构的 head 指针和 tail 指针， 程序获取链表的表头节点和表尾节点的复杂度为 O(1) 。
* 带链表长度计数器： 程序使用 list 结构的 len 属性来对 list 持有的链表节点进行计数， 程序获取链表中节点数量的复杂度为 O(1) 。
* 多态： 链表节点使用 void* 指针来保存节点值， 并且可以通过 list 结构的 dup 、 free 、 match 三个属性为节点值设置类型特定函数， 所以链表可以用于保存各种不同类型的值。

参考 http://redisbook.com