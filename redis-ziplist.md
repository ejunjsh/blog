---
title: redis数据结构-压缩列表
date: 2017-09-17 22:07:30
tags: [redis,数据结构,压缩列表]
categories: redis
---
> 压缩列表（ziplist）是列表键和哈希键的底层实现之一。
> 当一个列表键只包含少量列表项， 并且每个列表项要么就是小整数值， 要么就是长度比较短的字符串， 那么 Redis 就会使用压缩列表来做列表键的底层实现。

# 压缩列表的构成
压缩列表是 Redis 为了节约内存而开发的， 由一系列特殊编码的连续内存块组成的顺序型（sequential）数据结构。
一个压缩列表可以包含任意多个节点（entry）， 每个节点可以保存一个字节数组或者一个整数值。
图 7-1 展示了压缩列表的各个组成部分， 表 7-1 则记录了各个组成部分的类型、长度、以及用途。
[![](http://idiotsky.me/images1/redis-ziplist-1.png)](http://idiotsky.me/images1/redis-ziplist-1.png)

表 7-1 压缩列表各个组成部分的详细说明

属性|	类型	|长度	|用途
----|-----|----|----
zlbytes	|uint32_t	|4 字节	|记录整个压缩列表占用的内存字节数：在对压缩列表进行内存重分配， 或者计算 zlend 的位置时使用。
zltail|	uint32_t	|4 字节|	记录压缩列表表尾节点距离压缩列表的起始地址有多少字节： 通过这个偏移量，程序无须遍历整个压缩列表就可以确定表尾节点的地址。
zllen	|uint16_t	|2 字节	|记录了压缩列表包含的节点数量： 当这个属性的值小于 UINT16_MAX （65535）时， 这个属性的值就是压缩列表包含节点的数量； 当这个值等于 UINT16_MAX 时， 节点的真实数量需要遍历整个压缩列表才能计算得出。
entryX|	列表节点	|不定|	压缩列表包含的各个节点，节点的长度由节点保存的内容决定。
zlend|	uint8_t|	1 字节|	特殊值 0xFF （十进制 255 ），用于标记压缩列表的末端。

<!-- more -->
图 7-2 展示了一个压缩列表示例：
* 列表 zlbytes 属性的值为 0x50 （十进制 80）， 表示压缩列表的总长为 80 字节。
* 列表 zltail 属性的值为 0x3c （十进制 60）， 这表示如果我们有一个指向压缩列表起始地址的指针 p ， 那么只要用指针 p 加上偏移量 60 ， 就可以计算出表尾节点 entry3 的地址。
* 列表 zllen 属性的值为 0x3 （十进制 3）， 表示压缩列表包含三个节点。

[![](http://idiotsky.me/images1/redis-ziplist-2.png)](http://idiotsky.me/images1/redis-ziplist-2.png)
图 7-3 展示了另一个压缩列表示例：
* 列表 zlbytes 属性的值为 0xd2 （十进制 210）， 表示压缩列表的总长为 210 字节。
* 列表 zltail 属性的值为 0xb3 （十进制 179）， 这表示如果我们有一个指向压缩列表起始地址的指针 p ， 那么只要用指针 p 加上偏移量 179 ， 就可以计算出表尾节点 entry5 的地址。
* 列表 zllen 属性的值为 0x5 （十进制 5）， 表示压缩列表包含五个节点。

[![](http://idiotsky.me/images1/redis-ziplist-3.png)](http://idiotsky.me/images1/redis-ziplist-3.png)

# 压缩列表节点的构成
每个压缩列表节点可以保存一个字节数组或者一个整数值， 其中， 字节数组可以是以下三种长度的其中一种：
1. 长度小于等于 63 （2^{6}-1）字节的字节数组；
2. 长度小于等于 16383 （2^{14}-1） 字节的字节数组；
3. 长度小于等于 4294967295 （2^{32}-1）字节的字节数组；

而整数值则可以是以下六种长度的其中一种：
1. 4 位长，介于 0 至 12 之间的无符号整数；
2. 1 字节长的有符号整数；
3. 3 字节长的有符号整数；
4. int16_t 类型整数；
5. int32_t 类型整数；
6. int64_t 类型整数。

每个压缩列表节点都由 previous\_entry\_length 、 encoding 、 content 三个部分组成， 如图 7-4 所示。
[![](http://idiotsky.me/images1/redis-ziplist-4.png)](http://idiotsky.me/images1/redis-ziplist-4.png)
接下来的内容将分别介绍这三个组成部分。

## previous\_entry\_length
节点的 previous\_entry\_length 属性以字节为单位， 记录了压缩列表中前一个节点的长度。

previous\_entry\_length 属性的长度可以是 1 字节或者 5 字节：
* 如果前一节点的长度小于 254 字节， 那么 previous\_entry\_length 属性的长度为 1 字节： 前一节点的长度就保存在这一个字节里面。
* 如果前一节点的长度大于等于 254 字节， 那么 previous\_entry\_length 属性的长度为 5 字节： 其中属性的第一字节会被设置为 0xFE （十进制值 254）， 而之后的四个字节则用于保存前一节点的长度。

图 7-5 展示了一个包含一字节长 previous\_entry\_length 属性的压缩列表节点， 属性的值为 0x05 ， 表示前一节点的长度为 5 字节。
[![](http://idiotsky.me/images1/redis-ziplist-5.png)](http://idiotsky.me/images1/redis-ziplist-5.png)
图 7-6 展示了一个包含五字节长 previous\_entry\_length 属性的压缩节点， 属性的值为 0xFE00002766 ， 其中值的最高位字节 0xFE 表示这是一个五字节长的 previous\_entry\_length 属性， 而之后的四字节 0x00002766 （十进制值 10086 ）才是前一节点的实际长度。
[![](http://idiotsky.me/images1/redis-ziplist-6.png)](http://idiotsky.me/images1/redis-ziplist-6.png)

因为节点的 previous\_entry\_length 属性记录了前一个节点的长度， 所以程序可以通过指针运算， 根据当前节点的起始地址来计算出前一个节点的起始地址。

举个例子， 如果我们有一个指向当前节点起始地址的指针 c ， 那么我们只要用指针 c 减去当前节点 previous\_entry\_length 属性的值， 就可以得出一个指向前一个节点起始地址的指针 p ， 如图 7-7 所示。
[![](http://idiotsky.me/images1/redis-ziplist-7.png)](http://idiotsky.me/images1/redis-ziplist-7.png)

压缩列表的从表尾向表头遍历操作就是使用这一原理实现的： 只要我们拥有了一个指向某个节点起始地址的指针， 那么通过这个指针以及这个节点的 previous\_entry\_length 属性， 程序就可以一直向前一个节点回溯， 最终到达压缩列表的表头节点。

图 7-8 展示了一个从表尾节点向表头节点进行遍历的完整过程：
* 首先，我们拥有指向压缩列表表尾节点 entry4 起始地址的指针 p1 （指向表尾节点的指针可以通过指向压缩列表起始地址的指针加上 zltail 属性的值得出）；
* 通过用 p1 减去 entry4 节点 previous\_entry\_length 属性的值， 我们得到一个指向 entry4 前一节点 entry3 起始地址的指针 p2 ；
* 通过用 p2 减去 entry3 节点 previous\_entry\_length 属性的值， 我们得到一个指向 entry3 前一节点 entry2 起始地址的指针 p3 ；
* 通过用 p3 减去 entry2 节点 previous\_entry\_length 属性的值， 我们得到一个指向 entry2 前一节点 entry1 起始地址的指针 p4 ， entry1 为压缩列表的表头节点；
* 最终， 我们从表尾节点向表头节点遍历了整个列表。

[![](http://idiotsky.me/images1/redis-ziplist-8.png)](http://idiotsky.me/images1/redis-ziplist-8.png)
[![](http://idiotsky.me/images1/redis-ziplist-9.png)](http://idiotsky.me/images1/redis-ziplist-9.png)
[![](http://idiotsky.me/images1/redis-ziplist-10.png)](http://idiotsky.me/images1/redis-ziplist-10.png)
[![](http://idiotsky.me/images1/redis-ziplist-11.png)](http://idiotsky.me/images1/redis-ziplist-11.png)

## encoding
节点的 encoding 属性记录了节点的 content 属性所保存数据的类型以及长度：
* 一字节、两字节或者五字节长， 值的最高位为 00 、 01 或者 10 的是字节数组编码： 这种编码表示节点的 content 属性保存着字节数组， 数组的长度由编码除去最高两位之后的其他位记录；
* 一字节长， 值的最高位以 11 开头的是整数编码： 这种编码表示节点的 content 属性保存着整数值， 整数值的类型和长度由编码除去最高两位之后的其他位记录；

表 7-2 记录了所有可用的字节数组编码， 而表 7-3 则记录了所有可用的整数编码。 表格中的下划线 _ 表示留空， 而 b 、 x 等变量则代表实际的二进制数据， 为了方便阅读， 多个字节之间用空格隔开。

表 7-2 字节数组编码

编码|	编码长度	|content 属性保存的值
----|-------|-------|------
00bbbbbb|	1 字节	|长度小于等于 63 字节的字节数组。
01bbbbbb xxxxxxxx	|2 字节	|长度小于等于 16383 字节的字节数组。
10______ aaaaaaaa bbbbbbbb cccccccc dddddddd	|5 字节|	长度小于等于 4294967295 的字节数组。

表 7-3 整数编码

编码	|编码长度	|content 属性保存的值
-----|------|-------
11000000|	1 字节|	int16_t 类型的整数。
11010000|	1 字节|	int32_t 类型的整数。
11100000|	1 字节|	int64_t 类型的整数。
11110000|	1 字节|	24 位有符号整数。
11111110|	1 字节|	8 位有符号整数。
1111xxxx|	1 字节|	使用这一编码的节点没有相应的 content 属性， 因为编码本身的 xxxx 四个位已经保存了一个介于 0 和 12 之间的值， 所以它无须 content 属性。

## content
节点的 content 属性负责保存节点的值， 节点值可以是一个字节数组或者整数， 值的类型和长度由节点的 encoding 属性决定。

图 7-9 展示了一个保存字节数组的节点示例：
* 编码的最高两位 00 表示节点保存的是一个字节数组；
* 编码的后六位 001011 记录了字节数组的长度 11 ；
* content 属性保存着节点的值 "hello world" 。

[![](http://idiotsky.me/images1/redis-ziplist-12.png)](http://idiotsky.me/images1/redis-ziplist-12.png)

图 7-10 展示了一个保存整数值的节点示例：
* 编码 11000000 表示节点保存的是一个 int16_t 类型的整数值；
* content 属性保存着节点的值 10086 。

[![](http://idiotsky.me/images1/redis-ziplist-13.png)](http://idiotsky.me/images1/redis-ziplist-13.png)