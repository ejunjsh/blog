---
title: redis数据结构-对象
date: 2017-09-18 1:29:58
tags: [redis,数据结构]
categories: redis
---
# 前言
在前面的数个章节里， 我们陆续介绍了 Redis 用到的所有主要数据结构， 比如简单动态字符串（SDS）、双端链表、字典、压缩列表、整数集合， 等等。

Redis 并没有直接使用这些数据结构来实现键值对数据库， 而是基于这些数据结构创建了一个对象系统， 这个系统包含字符串对象、列表对象、哈希对象、集合对象和有序集合对象这五种类型的对象， 每种对象都用到了至少一种我们前面所介绍的数据结构。

通过这五种不同类型的对象， Redis 可以在执行命令之前， 根据对象的类型来判断一个对象是否可以执行给定的命令。 使用对象的另一个好处是， 我们可以针对不同的使用场景， 为对象设置多种不同的数据结构实现， 从而优化对象在不同场景下的使用效率。

除此之外， Redis 的对象系统还实现了基于引用计数技术的内存回收机制： 当程序不再使用某个对象的时候， 这个对象所占用的内存就会被自动释放； 另外， Redis 还通过引用计数技术实现了对象共享机制， 这一机制可以在适当的条件下， 通过让多个数据库键共享同一个对象来节约内存。

最后， Redis 的对象带有访问时间记录信息， 该信息可以用于计算数据库键的空转时长， 在服务器启用了 maxmemory 功能的情况下， 空转时长较大的那些键可能会优先被服务器删除。

本章接下来将逐一介绍以上提到的 Redis 对象系统的各个特性。
<!-- more -->
# 对象的类型与编码
Redis 使用对象来表示数据库中的键和值， 每次当我们在 Redis 的数据库中新创建一个键值对时， 我们至少会创建两个对象， 一个对象用作键值对的键（键对象）， 另一个对象用作键值对的值（值对象）。

举个例子， 以下 SET 命令在数据库中创建了一个新的键值对， 其中键值对的键是一个包含了字符串值 "msg" 的对象， 而键值对的值则是一个包含了字符串值 "hello world" 的对象：
````shell
redis> SET msg "hello world"
OK
````
Redis 中的每个对象都由一个 redisObject 结构表示， 该结构中和保存数据有关的三个属性分别是 type 属性、 encoding 属性和 ptr 属性：
````c
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 编码
    unsigned encoding:4;

    // 指向底层实现数据结构的指针
    void *ptr;

    // ...

} robj;
````

## 类型
对象的 type 属性记录了对象的类型， 这个属性的值可以是表 8-1 列出的常量的其中一个。

类型常量	|对象的名称
-------|----------
REDIS_STRING|	字符串对象
REDIS_LIST|	列表对象
REDIS_HASH|	哈希对象
REDIS_SET	|集合对象
REDIS_ZSET|	有序集合对象

对于 Redis 数据库保存的键值对来说， 键总是一个字符串对象， 而值则可以是字符串对象、列表对象、哈希对象、集合对象或者有序集合对象的其中一种， 因此：
* 当我们称呼一个数据库键为“字符串键”时， 我们指的是“这个数据库键所对应的值为字符串对象”；
* 当我们称呼一个键为“列表键”时， 我们指的是“这个数据库键所对应的值为列表对象”，

诸如此类。

TYPE 命令的实现方式也与此类似， 当我们对一个数据库键执行 TYPE 命令时， 命令返回的结果为数据库键对应的值对象的类型， 而不是键对象的类型：
````shell
# 键为字符串对象，值为字符串对象

redis> SET msg "hello world"
OK

redis> TYPE msg
string

# 键为字符串对象，值为列表对象

redis> RPUSH numbers 1 3 5
(integer) 6

redis> TYPE numbers
list

# 键为字符串对象，值为哈希对象

redis> HMSET profile name Tome age 25 career Programmer
OK

redis> TYPE profile
hash

# 键为字符串对象，值为集合对象

redis> SADD fruits apple banana cherry
(integer) 3

redis> TYPE fruits
set

# 键为字符串对象，值为有序集合对象

redis> ZADD price 8.5 apple 5.0 banana 6.0 cherry
(integer) 3

redis> TYPE price
zset
````
表 8-2 列出了 TYPE 命令在面对不同类型的值对象时所产生的输出。

对象|	对象 type 属性的值|	TYPE 命令的输出
----|---------|-------
字符串对象|	REDIS_STRING|	"string"
列表对象|	REDIS_LIST	|"list"
哈希对象|	REDIS_HASH	|"hash"
集合对象|	REDIS_SET	|"set"
有序集合对象|	REDIS_ZSET	|"zset"

## 编码和底层实现
对象的 ptr 指针指向对象的底层实现数据结构， 而这些数据结构由对象的 encoding 属性决定。

encoding 属性记录了对象所使用的编码， 也即是说这个对象使用了什么数据结构作为对象的底层实现， 这个属性的值可以是表 8-3 列出的常量的其中一个。

编码常量	|编码所对应的底层数据结构
------|-----------
REDIS_ENCODING_INT	|long 类型的整数
REDIS_ENCODING_EMBSTR	|embstr 编码的简单动态字符串
REDIS_ENCODING_RAW	|简单动态字符串
REDIS_ENCODING_HT|	字典
REDIS_ENCODING_LINKEDLIST	|双端链表
REDIS_ENCODING_ZIPLIST|	压缩列表
REDIS_ENCODING_INTSET	|整数集合
REDIS_ENCODING_SKIPLIST	|跳跃表和字典

每种类型的对象都至少使用了两种不同的编码， 表 8-4 列出了每种类型的对象可以使用的编码。

类型	|编码	|对象
------|-----|----
REDIS_STRING	|REDIS_ENCODING_INT|	使用整数值实现的字符串对象。
REDIS_STRING|	REDIS_ENCODING_EMBSTR|	使用 embstr 编码的简单动态字符串实现的字符串对象。
REDIS_STRING	|REDIS_ENCODING_RAW	|使用简单动态字符串实现的字符串对象。
REDIS_LIST|	REDIS_ENCODING_ZIPLIST|	使用压缩列表实现的列表对象。
REDIS_LIST|	REDIS_ENCODING_LINKEDLIST	|使用双端链表实现的列表对象。
REDIS_HASH|	REDIS_ENCODING_ZIPLIST|	使用压缩列表实现的哈希对象。
REDIS_HASH|	REDIS_ENCODING_HT	|使用字典实现的哈希对象。
REDIS_SET	|REDIS_ENCODING_INTSET	|使用整数集合实现的集合对象。
REDIS_SET	|REDIS_ENCODING_HT	|使用字典实现的集合对象。
REDIS_ZSET|	REDIS_ENCODING_ZIPLIST|	使用压缩列表实现的有序集合对象。
REDIS_ZSET|	REDIS_ENCODING_SKIPLIST|	使用跳跃表和字典实现的有序集合对象。

使用 OBJECT ENCODING 命令可以查看一个数据库键的值对象的编码：
````shell
redis> SET msg "hello wrold"
OK

redis> OBJECT ENCODING msg
"embstr"

redis> SET story "long long long long long long ago ..."
OK

redis> OBJECT ENCODING story
"raw"

redis> SADD numbers 1 3 5
(integer) 3

redis> OBJECT ENCODING numbers
"intset"

redis> SADD numbers "seven"
(integer) 1

redis> OBJECT ENCODING numbers
"hashtable"
````
表 8-5 列出了不同编码的对象所对应的 OBJECT ENCODING 命令输出。

对象所使用的底层数据结构	|编码常量|	OBJECT ENCODING 命令输出
------|-----|-------
整数|	REDIS_ENCODING_INT|	"int"
embstr 编码的简单动态字符串（SDS）	|REDIS_ENCODING_EMBSTR|	"embstr"
简单动态字符串|	REDIS_ENCODING_RAW|	"raw"
字典|	REDIS_ENCODING_HT|	"hashtable"
双端链表|	REDIS_ENCODING_LINKEDLIST|	"linkedlist"
压缩列表|	REDIS_ENCODING_ZIPLIST|	"ziplist"
整数集合|	REDIS_ENCODING_INTSET|	"intset"
跳跃表和字典|	REDIS_ENCODING_SKIPLIST|	"skiplist"

通过 encoding 属性来设定对象所使用的编码， 而不是为特定类型的对象关联一种固定的编码， 极大地提升了 Redis 的灵活性和效率， 因为 Redis 可以根据不同的使用场景来为一个对象设置不同的编码， 从而优化对象在某一场景下的效率。

举个例子， 在列表对象包含的元素比较少时， Redis 使用压缩列表作为列表对象的底层实现：
* 因为压缩列表比双端链表更节约内存， 并且在元素数量较少时， 在内存中以连续块方式保存的压缩列表比起双端链表可以更快被载入到缓存中；
* 随着列表对象包含的元素越来越多， 使用压缩列表来保存元素的优势逐渐消失时， 对象就会将底层实现从压缩列表转向功能更强、也更适合保存大量元素的双端链表上面；

其他类型的对象也会通过使用多种不同的编码来进行类似的优化。

在接下来的内容中， 我们将分别介绍 Redis 中的五种不同类型的对象， 说明这些对象底层所使用的编码方式， 列出对象从一种编码转换成另一种编码所需的条件， 以及同一个命令在多种不同编码上的实现方法。

# 字符串对象
字符串对象的编码可以是 int 、 raw 或者 embstr 。

如果一个字符串对象保存的是整数值， 并且这个整数值可以用 long 类型来表示， 那么字符串对象会将整数值保存在字符串对象结构的 ptr 属性里面（将 void* 转换成 long ）， 并将字符串对象的编码设置为 int 。

举个例子， 如果我们执行以下 SET 命令， 那么服务器将创建一个如图 8-1 所示的 int 编码的字符串对象作为 number 键的值：
````shell
redis> SET number 10086
OK

redis> OBJECT ENCODING number
"int"
````
[![](http://idiotsky.me/images1/redis-object-1.png)](http://idiotsky.me/images1/redis-object-1.png)
如果字符串对象保存的是一个字符串值， 并且这个字符串值的长度大于 39 字节， 那么字符串对象将使用一个简单动态字符串（SDS）来保存这个字符串值， 并将对象的编码设置为 raw 。

举个例子， 如果我们执行以下命令， 那么服务器将创建一个如图 8-2 所示的 raw 编码的字符串对象作为 story 键的值：
````shell
redis> SET story "Long, long, long ago there lived a king ..."
OK

redis> STRLEN story
(integer) 43

redis> OBJECT ENCODING story
"raw"
````
[![](http://idiotsky.me/images1/redis-object-2.png)](http://idiotsky.me/images1/redis-object-2.png)
如果字符串对象保存的是一个字符串值， 并且这个字符串值的长度小于等于 39 字节， 那么字符串对象将使用 embstr 编码的方式来保存这个字符串值。

embstr 编码是专门用于保存短字符串的一种优化编码方式， 这种编码和 raw 编码一样， 都使用 redisObject 结构和 sdshdr 结构来表示字符串对象， 但 raw 编码会调用两次内存分配函数来分别创建 redisObject 结构和 sdshdr 结构， 而 embstr 编码则通过调用一次内存分配函数来分配一块连续的空间， 空间中依次包含 redisObject 和 sdshdr 两个结构， 如图 8-3 所示。
[![](http://idiotsky.me/images1/redis-object-3.png)](http://idiotsky.me/images1/redis-object-3.png)
embstr 编码的字符串对象在执行命令时， 产生的效果和 raw 编码的字符串对象执行命令时产生的效果是相同的， 但使用 embstr 编码的字符串对象来保存短字符串值有以下好处：

embstr 编码将创建字符串对象所需的内存分配次数从 raw 编码的两次降低为一次。
释放 embstr 编码的字符串对象只需要调用一次内存释放函数， 而释放 raw 编码的字符串对象需要调用两次内存释放函数。
因为 embstr 编码的字符串对象的所有数据都保存在一块连续的内存里面， 所以这种编码的字符串对象比起 raw 编码的字符串对象能够更好地利用缓存带来的优势。
作为例子， 以下命令创建了一个 embstr 编码的字符串对象作为 msg 键的值， 值对象的样子如图 8-4 所示：
````shell
redis> SET msg "hello"
OK

redis> OBJECT ENCODING msg
"embstr"
````
[![](http://idiotsky.me/images1/redis-object-4.png)](http://idiotsky.me/images1/redis-object-4.png)
最后要说的是， 可以用 long double 类型表示的浮点数在 Redis 中也是作为字符串值来保存的： 如果我们要保存一个浮点数到字符串对象里面， 那么程序会先将这个浮点数转换成字符串值， 然后再保存起转换所得的字符串值。

举个例子， 执行以下代码将创建一个包含 3.14 的字符串表示 "3.14" 的字符串对象：
````shell
redis> SET pi 3.14
OK

redis> OBJECT ENCODING pi
"embstr"
````
在有需要的时候， 程序会将保存在字符串对象里面的字符串值转换回浮点数值， 执行某些操作， 然后再将执行操作所得的浮点数值转换回字符串值， 并继续保存在字符串对象里面。

举个例子， 如果我们执行以下代码的话：
````shell
redis> INCRBYFLOAT pi 2.0
"5.14"

redis> OBJECT ENCODING pi
"embstr"
````
那么程序首先会取出字符串对象里面保存的字符串值 "3.14" ， 将它转换回浮点数值 3.14 ， 然后把 3.14 和 2.0 相加得出的值 5.14 转换成字符串 "5.14" ， 并将这个 "5.14" 保存到字符串对象里面。

表 8-6 总结并列出了字符串对象保存各种不同类型的值所使用的编码方式。

值	|编码
-----|-----
可以用 long 类型保存的整数。	|int
可以用 long double 类型保存的浮点数。	|embstr 或者 raw
字符串值， 或者因为长度太大而没办法用 long 类型表示的整数， 又或者因为长度太大而没办法用 long double 类型表示的浮点数。|	embstr 或者 raw

## 编码的转换
int 编码的字符串对象和 embstr 编码的字符串对象在条件满足的情况下， 会被转换为 raw 编码的字符串对象。

对于 int 编码的字符串对象来说， 如果我们向对象执行了一些命令， 使得这个对象保存的不再是整数值， 而是一个字符串值， 那么字符串对象的编码将从 int 变为 raw 。

在下面的示例中， 我们通过 APPEND 命令， 向一个保存整数值的字符串对象追加了一个字符串值， 因为追加操作只能对字符串值执行， 所以程序会先将之前保存的整数值 10086 转换为字符串值 "10086" ， 然后再执行追加操作， 操作的执行结果就是一个 raw 编码的、保存了字符串值的字符串对象：
````shell
redis> SET number 10086
OK

redis> OBJECT ENCODING number
"int"

redis> APPEND number " is a good number!"
(integer) 23

redis> GET number
"10086 is a good number!"

redis> OBJECT ENCODING number
"raw"
````
另外， 因为 Redis 没有为 embstr 编码的字符串对象编写任何相应的修改程序 （只有 int 编码的字符串对象和 raw 编码的字符串对象有这些程序）， 所以 embstr 编码的字符串对象实际上是只读的： 当我们对 embstr 编码的字符串对象执行任何修改命令时， 程序会先将对象的编码从 embstr 转换成 raw ， 然后再执行修改命令； 因为这个原因， embstr 编码的字符串对象在执行修改命令之后， 总会变成一个 raw 编码的字符串对象。

以下代码展示了一个 embstr 编码的字符串对象在执行 APPEND 命令之后， 对象的编码从 embstr 变为 raw 的例子：
````shell
redis> SET msg "hello world"
OK

redis> OBJECT ENCODING msg
"embstr"

redis> APPEND msg " again!"
(integer) 18

redis> OBJECT ENCODING msg
"raw"
````

# 列表对象
> to be continue...