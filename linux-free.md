---
title: Linux的free命令详解
date: 2014-06-10 10:21:13
tags: [linux,free]
categories: linux命令
---
# 属性讲解
执行下面命令
````shell
$ free
````
结果(默认单位为KB)
````
               total       used       free     shared    buffers     cached
  Mem:      24677460   23276064    1401396          0     870540   12084008
  -/+ buffers/cache:   10321516   14355944
  Swap:     25151484     224188   24927296
````
<!-- more -->
Mem：表示物理内存统计
````
total：表示物理内存总量(total = used + free)
used：表示总计分配给缓存（包含buffers 与cached ）使用的数量，但其中可能部分缓存并未实际使用。
free：未被分配的内存。
shared：共享内存，一般系统不会用到，这里也不讨论。
buffers：系统分配但未被使用的buffers 数量。
cached：系统分配但未被使用的cache 数量。
````
-/+ buffers/cache：表示物理内存的缓存统计
````
used2：也就是第一行中的used – buffers-cached 也是实际使用的内存总量。 //used2为第二行
free2= buffers1 + cached1 + free1 //free2为第二行、buffers1等为第一行
free2：未被使用的buffers 与cache 和未被分配的内存之和，这就是系统当前实际可用内存。 
````
Swap：表示硬盘上交换分区的使用情况，这里我们不去关心。
系统的总物理内存：24677460（240M），但系统当前真正可用的内存并不是第一行free 标记的 1401396Kb，它仅代表未被分配的内存。

# buffers与cached的区别
A buffer is something that has yet to be “written” to disk. A cache is something that has been “read” from the disk and stored for later use 

对于应用程序来说，buffers/cached 是等于可用的，因为buffer/cached是为了提高文件读取的性能，当应用程序需在用到内存的时候，buffer/cached会很快地被回收。
所以从应用程序的角度来说**可用内存=系统free memory+buffers+cached**.

buffers是指用来给块设备做的缓冲大小，他只记录文件系统的metadata以及 tracking in-flight pages.
cached是用来给文件做缓冲。
那就是说：buffers是用来存储，目录里面有什么内容，权限等等。
而cached直接用来记忆我们打开的文件，如果你想知道他是不是真的生效，你可以试一下，先后执行两次命令#man X ,你就可以明显的感觉到第二次的开打的速度快很多。

## cached实验
在一台没有什么应用的机器上做会看得比较明显。记得实验只能做一次，如果想多做请换一个文件名。
````
#free
#man X
#free
#man X
#free
````
你可以先后比较一下free后显示cached的大小。

## buffers实验
````
#free
#ls /dev
#free
````
你比较一下两个的大小，当然这个buffers随时都在增加，但你有ls过的话，增加的速度会变得快，这个就是buffers/chached的区别。

因为Linux将你暂时不使用的内存作为文件和数据缓存，以提高系统性能，当你需要这些内存时，系统会自动释放（不像windows那样，即使你有很多空闲内存,他也要访问一下磁盘中的pagefiles）

# 简述swap
当可用内存少于额定值的时候，就会开始进行交换.
如何看额定值（RHEL4.0）：
````
#cat /proc/meminfo
````
交换将通过三个途径来减少系统中使用的物理页面的个数：
1. 减少缓冲与页面cache的大小，
2. 将系统V类型的内存页面交换出去，
3. 换出或者丢弃页面。(Application 占用的内存页，也就是物理内存不足）。

事实上，少量地使用swap是不是影响到系统性能的。

# 使用free命令
````
语　　法： free [-bkmotV][-s <间隔秒数>]

补充说明：free指令会显示内存的使用情况，包括实体内存，虚拟的交换文件内存，共享内存区段，以及系统核心使用的缓冲区等。

参　　数：
-b 　以Byte为单位显示内存使用情况。
-k 　以KB为单位显示内存使用情况。
-m 　以MB为单位显示内存使用情况。
-o 　不显示缓冲区调节列。
-s<间隔秒数> 　持续观察内存使用状况。
-t 　显示内存总和列。
-V 　显示版本信息。
````
常用操作:
````
free //以KB为单位，显式系统内存使用情况
free -ml -s 1  //每秒以M为单位，显式系统内存详细使用情况。
free -c 4 -s 2  //为KB为单位，每2秒显式系统内存使用情况，一共显示4次
````

# 总结
将used的值减去buffer和cache的值就是你当前真实内存使用--对操作系统来讲是Mem的参数.buffers/cached 都是属于被使用,所以它认为free只有1401396.
对应用程序来讲是(-/+ buffers/cached).buffers/cached 是等同可用的，因为buffer/cached是为了提高 程序执行的性能，当程序使用内存时，buffer/cached会很快地被使用。 所以,以应用来看看,以(-/+ buffers/cache)的free和used为主.所以我们看这个就好了.另外告诉大家 一些常识.Linux为了提高磁盘和内存存取效率, Linux做了很多精心的设计, 除了对dentry进行缓存(用于 VFS,加速文件路径名到inode的转换), 还采取了两种主要Cache方式：Buffer Cache和Page Cache。 前者针对磁盘块的读写，后者针对文件inode的读写。这些Cache能有效缩短了 I/O系统调用(比如read,write,getdents)的时间。 记住内存是拿来用的,不是拿来看的.不像windows,无论你的真实物理内存有多少,他都要拿硬盘交换 文件来读.这也就是windows为什么常常提示虚拟空间不足的原因.你们想想,多无聊,在内存还有大部分 的时候,拿出一部分硬盘空间来充当内存.硬盘怎么会快过内存.所以我们看linux,只要不用swap的交换 空间,就不用担心自己的内存太少.如果常常swap用很多,可能你就要考虑加物理内存了.这也是linux看 内存是否够用的标准哦.

参考 http://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316438.html