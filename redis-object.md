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
列表对象的编码可以是 ziplist 或者 linkedlist 。

ziplist 编码的列表对象使用压缩列表作为底层实现， 每个压缩列表节点（entry）保存了一个列表元素。

举个例子， 如果我们执行以下 RPUSH 命令， 那么服务器将创建一个列表对象作为 numbers 键的值：
````shell
redis> RPUSH numbers 1 "three" 5
(integer) 3
````
如果 numbers 键的值对象使用的是 ziplist 编码， 这个这个值对象将会是图 8-5 所展示的样子。
[![](http://idiotsky.me/images1/redis-object-5.png)](http://idiotsky.me/images1/redis-object-5.png)
另一方面， linkedlist 编码的列表对象使用双端链表作为底层实现， 每个双端链表节点（node）都保存了一个字符串对象， 而每个字符串对象都保存了一个列表元素。

举个例子， 如果前面所说的 numbers 键创建的列表对象使用的不是 ziplist 编码， 而是 linkedlist 编码， 那么 numbers 键的值对象将是图 8-6 所示的样子。
[![](http://idiotsky.me/images1/redis-object-6.png)](http://idiotsky.me/images1/redis-object-6.png)
注意， linkedlist 编码的列表对象在底层的双端链表结构中包含了多个字符串对象， 这种嵌套字符串对象的行为在稍后介绍的哈希对象、集合对象和有序集合对象中都会出现， 字符串对象是 Redis 五种类型的对象中唯一一种会被其他四种类型对象嵌套的对象。

## 编码转换
当列表对象可以同时满足以下两个条件时， 列表对象使用 ziplist 编码：
1. 列表对象保存的所有字符串元素的长度都小于 64 字节；
2. 列表对象保存的元素数量小于 512 个；

不能满足这两个条件的列表对象需要使用 linkedlist 编码。

>__注意__
>以上两个条件的上限值是可以修改的， 具体请看配置文件中关于 list-max-ziplist-value 选项和 list-max-ziplist-entries 选项的说明。

对于使用 ziplist 编码的列表对象来说， 当使用 ziplist 编码所需的两个条件的任意一个不能被满足时， 对象的编码转换操作就会被执行： 原本保存在压缩列表里的所有列表元素都会被转移并保存到双端链表里面， 对象的编码也会从 ziplist 变为 linkedlist 。

以下代码展示了列表对象因为保存了长度太大的元素而进行编码转换的情况：
````shell
# 所有元素的长度都小于 64 字节
redis> RPUSH blah "hello" "world" "again"
(integer) 3

redis> OBJECT ENCODING blah
"ziplist"

# 将一个 65 字节长的元素推入列表对象中
redis> RPUSH blah "wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwww"
(integer) 4

# 编码已改变
redis> OBJECT ENCODING blah
"linkedlist"
````
除此之外， 以下代码展示了列表对象因为保存的元素数量过多而进行编码转换的情况：
````shell
# 列表对象包含 512 个元素
redis> EVAL "for i=1,512 do redis.call('RPUSH', KEYS[1], i) end" 1 "integers"
(nil)

redis> LLEN integers
(integer) 512

redis> OBJECT ENCODING integers
"ziplist"

# 再向列表对象推入一个新元素，使得对象保存的元素数量达到 513 个
redis> RPUSH integers 513
(integer) 513

# 编码已改变
redis> OBJECT ENCODING integers
"linkedlist"
````

# 哈希对象
哈希对象的编码可以是 ziplist 或者 hashtable 。

ziplist 编码的哈希对象使用压缩列表作为底层实现， 每当有新的键值对要加入到哈希对象时， 程序会先将保存了键的压缩列表节点推入到压缩列表表尾， 然后再将保存了值的压缩列表节点推入到压缩列表表尾， 因此：
* 保存了同一键值对的两个节点总是紧挨在一起， 保存键的节点在前， 保存值的节点在后；
* 先添加到哈希对象中的键值对会被放在压缩列表的表头方向， 而后来添加到哈希对象中的键值对会被放在压缩列表的表尾方向。

举个例子， 如果我们执行以下 HSET 命令， 那么服务器将创建一个列表对象作为 profile 键的值：
````shell
redis> HSET profile name "Tom"
(integer) 1

redis> HSET profile age 25
(integer) 1

redis> HSET profile career "Programmer"
(integer) 1
````
如果 profile 键的值对象使用的是 ziplist 编码， 那么这个值对象将会是图 8-9 所示的样子， 其中对象所使用的压缩列表如图 8-10 所示。
[![](http://idiotsky.me/images1/redis-object-9.png)](http://idiotsky.me/images1/redis-object-9.png)
[![](http://idiotsky.me/images1/redis-object-10.png)](http://idiotsky.me/images1/redis-object-10.png)

另一方面， hashtable 编码的哈希对象使用字典作为底层实现， 哈希对象中的每个键值对都使用一个字典键值对来保存：
* 字典的每个键都是一个字符串对象， 对象中保存了键值对的键；
* 字典的每个值都是一个字符串对象， 对象中保存了键值对的值。

举个例子， 如果前面 profile 键创建的不是 ziplist 编码的哈希对象， 而是 hashtable 编码的哈希对象， 那么这个哈希对象应该会是图 8-11 所示的样子。
[![](http://idiotsky.me/images1/redis-object-11.png)](http://idiotsky.me/images1/redis-object-11.png)

## 编码转换
当哈希对象可以同时满足以下两个条件时， 哈希对象使用 ziplist 编码：
1. 哈希对象保存的所有键值对的键和值的字符串长度都小于 64 字节；
2. 哈希对象保存的键值对数量小于 512 个；

不能满足这两个条件的哈希对象需要使用 hashtable 编码。

>__注意__
>这两个条件的上限值是可以修改的， 具体请看配置文件中关于 hash-max-ziplist-value 选项和 hash-max-ziplist-entries 选项的说明。

对于使用 ziplist 编码的列表对象来说， 当使用 ziplist 编码所需的两个条件的任意一个不能被满足时， 对象的编码转换操作就会被执行： 原本保存在压缩列表里的所有键值对都会被转移并保存到字典里面， 对象的编码也会从 ziplist 变为 hashtable 。

以下代码展示了哈希对象因为键值对的键长度太大而引起编码转换的情况：
````shell
# 哈希对象只包含一个键和值都不超过 64 个字节的键值对
redis> HSET book name "Mastering C++ in 21 days"
(integer) 1

redis> OBJECT ENCODING book
"ziplist"

# 向哈希对象添加一个新的键值对，键的长度为 66 字节
redis> HSET book long_long_long_long_long_long_long_long_long_long_long_description "content"
(integer) 1

# 编码已改变
redis> OBJECT ENCODING book
"hashtable"
````
除了键的长度太大会引起编码转换之外， 值的长度太大也会引起编码转换， 以下代码展示了这种情况的一个示例：
````shell
# 哈希对象只包含一个键和值都不超过 64 个字节的键值对
redis> HSET blah greeting "hello world"
(integer) 1

redis> OBJECT ENCODING blah
"ziplist"

# 向哈希对象添加一个新的键值对，值的长度为 68 字节
redis> HSET blah story "many string ... many string ... many string ... many string ... many"
(integer) 1

# 编码已改变
redis> OBJECT ENCODING blah
"hashtable"
````
最后， 以下代码展示了哈希对象因为包含的键值对数量过多而引起编码转换的情况：
````shell
# 创建一个包含 512 个键值对的哈希对象
redis> EVAL "for i=1, 512 do redis.call('HSET', KEYS[1], i, i) end" 1 "numbers"
(nil)

redis> HLEN numbers
(integer) 512

redis> OBJECT ENCODING numbers
"ziplist"

# 再向哈希对象添加一个新的键值对，使得键值对的数量变成 513 个
redis> HMSET numbers "key" "value"
OK

redis> HLEN numbers
(integer) 513

# 编码改变
redis> OBJECT ENCODING numbers
"hashtable"
````

# 集合对象
集合对象的编码可以是 intset 或者 hashtable 。

intset 编码的集合对象使用整数集合作为底层实现， 集合对象包含的所有元素都被保存在整数集合里面。

举个例子， 以下代码将创建一个如图 8-12 所示的 intset 编码集合对象：
````shell
redis> SADD numbers 1 3 5
(integer) 3
````
[![](http://idiotsky.me/images1/redis-object-12.png)](http://idiotsky.me/images1/redis-object-12.png)
另一方面， hashtable 编码的集合对象使用字典作为底层实现， 字典的每个键都是一个字符串对象， 每个字符串对象包含了一个集合元素， 而字典的值则全部被设置为 NULL 。

举个例子， 以下代码将创建一个如图 8-13 所示的 hashtable 编码集合对象：
````shell
redis> SADD fruits "apple" "banana" "cherry"
(integer) 3
````
[![](http://idiotsky.me/images1/redis-object-13.png)](http://idiotsky.me/images1/redis-object-13.png)

## 编码的转换
当集合对象可以同时满足以下两个条件时， 对象使用 intset 编码：
1. 集合对象保存的所有元素都是整数值；
2. 集合对象保存的元素数量不超过 512 个；

不能满足这两个条件的集合对象需要使用 hashtable 编码。

>__注意__
>第二个条件的上限值是可以修改的， 具体请看配置文件中关于 set-max-intset-entries 选项的说明。

对于使用 intset 编码的集合对象来说， 当使用 intset 编码所需的两个条件的任意一个不能被满足时， 对象的编码转换操作就会被执行： 原本保存在整数集合中的所有元素都会被转移并保存到字典里面， 并且对象的编码也会从 intset 变为 hashtable 。

举个例子， 以下代码创建了一个只包含整数元素的集合对象， 该对象的编码为 intset ：
````shell
redis> SADD numbers 1 3 5
(integer) 3

redis> OBJECT ENCODING numbers
"intset"
````
不过， 只要我们向这个只包含整数元素的集合对象添加一个字符串元素， 集合对象的编码转移操作就会被执行：
````shell
redis> SADD numbers "seven"
(integer) 1

redis> OBJECT ENCODING numbers
"hashtable"
````
除此之外， 如果我们创建一个包含 512 个整数元素的集合对象， 那么对象的编码应该会是 intset ：
````shell
redis> EVAL "for i=1, 512 do redis.call('SADD', KEYS[1], i) end" 1 integers
(nil)

redis> SCARD integers
(integer) 512

redis> OBJECT ENCODING integers
"intset"
````
但是， 只要我们再向集合添加一个新的整数元素， 使得这个集合的元素数量变成 513 ， 那么对象的编码转换操作就会被执行：
````shell
redis> SADD integers 10086
(integer) 1

redis> SCARD integers
(integer) 513

redis> OBJECT ENCODING integers
"hashtable"
````

# 有序集合对象
有序集合的编码可以是 ziplist 或者 skiplist 。

ziplist 编码的有序集合对象使用压缩列表作为底层实现， 每个集合元素使用两个紧挨在一起的压缩列表节点来保存， 第一个节点保存元素的成员（member）， 而第二个元素则保存元素的分值（score）。

压缩列表内的集合元素按分值从小到大进行排序， 分值较小的元素被放置在靠近表头的方向， 而分值较大的元素则被放置在靠近表尾的方向。

举个例子， 如果我们执行以下 ZADD 命令， 那么服务器将创建一个有序集合对象作为 price 键的值：
````shell
redis> ZADD price 8.5 apple 5.0 banana 6.0 cherry
(integer) 3
````
如果 price 键的值对象使用的是 ziplist 编码， 那么这个值对象将会是图 8-14 所示的样子， 而对象所使用的压缩列表则会是 8-15 所示的样子。
[![](http://idiotsky.me/images1/redis-object-14.png)](http://idiotsky.me/images1/redis-object-14.png)
[![](http://idiotsky.me/images1/redis-object-15.png)](http://idiotsky.me/images1/redis-object-15.png)
skiplist 编码的有序集合对象使用 zset 结构作为底层实现， 一个 zset 结构同时包含一个字典和一个跳跃表：
````c
typedef struct zset {

    zskiplist *zsl;

    dict *dict;

} zset;
````
zset 结构中的 zsl 跳跃表按分值从小到大保存了所有集合元素， 每个跳跃表节点都保存了一个集合元素： 跳跃表节点的 object 属性保存了元素的成员， 而跳跃表节点的 score 属性则保存了元素的分值。 通过这个跳跃表， 程序可以对有序集合进行范围型操作， 比如 ZRANK 、 ZRANGE 等命令就是基于跳跃表 API 来实现的。

除此之外， zset 结构中的 dict 字典为有序集合创建了一个从成员到分值的映射， 字典中的每个键值对都保存了一个集合元素： 字典的键保存了元素的成员， 而字典的值则保存了元素的分值。 通过这个字典， 程序可以用 O(1) 复杂度查找给定成员的分值， ZSCORE 命令就是根据这一特性实现的， 而很多其他有序集合命令都在实现的内部用到了这一特性。

有序集合每个元素的成员都是一个字符串对象， 而每个元素的分值都是一个 double 类型的浮点数。 值得一提的是， 虽然 zset 结构同时使用跳跃表和字典来保存有序集合元素， 但这两种数据结构都会通过指针来共享相同元素的成员和分值， 所以同时使用跳跃表和字典来保存集合元素不会产生任何重复成员或者分值， 也不会因此而浪费额外的内存。

>__为什么有序集合需要同时使用跳跃表和字典来实现？__
>在理论上来说， 有序集合可以单独使用字典或者跳跃表的其中一种数据结构来实现， 但无论单独使用字典还是跳跃表， 在性能上对比起同时使用字典和跳跃表都会有所降低。

>举个例子， 如果我们只使用字典来实现有序集合， 那么虽然以 O(1) 复杂度查找成员的分值这一特性会被保留， 但是， 因为字典以无序的方式来保存集合元素， 所以每次在执行范围型操作 —— 比如 ZRANK 、 ZRANGE 等命令时， 程序都需要对字典保存的所有元素进行排序， 完成这种排序需要至少 O(N \log N) 时间复杂度， 以及额外的 O(N) 内存空间 （因为要创建一个数组来保存排序后的元素）。

>另一方面， 如果我们只使用跳跃表来实现有序集合， 那么跳跃表执行范围型操作的所有优点都会被保留， 但因为没有了字典， 所以根据成员查找分值这一操作的复杂度将从 O(1) 上升为 O(\log N) 。

>因为以上原因， 为了让有序集合的查找和范围型操作都尽可能快地执行， Redis 选择了同时使用字典和跳跃表两种数据结构来实现有序集合。

举个例子， 如果前面 price 键创建的不是 ziplist 编码的有序集合对象， 而是 skiplist 编码的有序集合对象， 那么这个有序集合对象将会是图 8-16 所示的样子， 而对象所使用的 zset 结构将会是图 8-17 所示的样子。
[![](http://idiotsky.me/images1/redis-object-16.png)](http://idiotsky.me/images1/redis-object-16.png)
[![](http://idiotsky.me/images1/redis-object-17.png)](http://idiotsky.me/images1/redis-object-17.png)

>__注意__
>为了展示方便， 图 8-17 在字典和跳跃表中重复展示了各个元素的成员和分值， 但在实际中， 字典和跳跃表会共享元素的成员和分值， 所以并不会造成任何数据重复， 也不会因此而浪费任何内存。

## 编码的转换
当有序集合对象可以同时满足以下两个条件时， 对象使用 ziplist 编码：
1. 有序集合保存的元素数量小于 128 个；
2. 有序集合保存的所有元素成员的长度都小于 64 字节；

不能满足以上两个条件的有序集合对象将使用 skiplist 编码。
>__注意__ 
>以上两个条件的上限值是可以修改的， 具体请看配置文件中关于 zset-max-ziplist-entries 选项和 zset-max-ziplist-value 选项的说明。

对于使用 ziplist 编码的有序集合对象来说， 当使用 ziplist 编码所需的两个条件中的任意一个不能被满足时， 程序就会执行编码转换操作， 将原本储存在压缩列表里面的所有集合元素转移到 zset 结构里面， 并将对象的编码从 ziplist 改为 skiplist 。

以下代码展示了有序集合对象因为包含了过多元素而引发编码转换的情况：
````shell
# 对象包含了 128 个元素
redis> EVAL "for i=1, 128 do redis.call('ZADD', KEYS[1], i, i) end" 1 numbers
(nil)

redis> ZCARD numbers
(integer) 128

redis> OBJECT ENCODING numbers
"ziplist"

# 再添加一个新元素
redis> ZADD numbers 3.14 pi
(integer) 1

# 对象包含的元素数量变为 129 个
redis> ZCARD numbers
(integer) 129

# 编码已改变
redis> OBJECT ENCODING numbers
"skiplist"
````
以下代码则展示了有序集合对象因为元素的成员过长而引发编码转换的情况：
````shell
# 向有序集合添加一个成员只有三字节长的元素
redis> ZADD blah 1.0 www
(integer) 1

redis> OBJECT ENCODING blah
"ziplist"

# 向有序集合添加一个成员为 66 字节长的元素
redis> ZADD blah 2.0 oooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo
(integer) 1

# 编码已改变
redis> OBJECT ENCODING blah
"skiplist"
````

# 重点回顾
* Redis 数据库中的每个键值对的键和值都是一个对象。
* Redis 共有字符串、列表、哈希、集合、有序集合五种类型的对象， 每种类型的对象至少都有两种或以上的编码方式， 不同的编码可以在不同的使用场景上优化对象的使用效率。

参考 http://redisbook.com