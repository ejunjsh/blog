---
title: 使用iostat分析IO性能
date: 2018-11-25 17:39:20
tags: [linux,iostat]
categories: linux命令
---

> iostat用于输出CPU和磁盘I/O相关的统计信息.👿

# 不加选项执行iostat

单独执行iostat，显示的结果为从系统开机到当前执行时刻的统计信息。

````
$ iostat
Linux 2.6.32-279.19.3.el6.ucloud.x86_64 (vm1)   06/11/2017  _x86_64_    (8 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.08    0.00    0.06    0.00    0.00   99.86

Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
vda               0.45         0.29         8.10    6634946  183036680
vdb               0.12         3.11        30.55   70342034  689955328

````

<!-- more -->

以上输出中，包含三部分：

* 第一行:最上面指示系统版本、主机名和当前日期
* avg-cpu:总体cpu使用情况统计信息，对于多核cpu，这里为所有cpu的平均值
* Device:各磁盘设备的IO统计信息

avg-cpu中各列参数含义如下：

* %user:CPU在用户态执行进程的时间百分比。
* %nice:CPU在用户态模式下，用于nice操作，所占用CPU总时间的百分比
* %system:CPU处在内核态执行进程的时间百分比
* %iowait:CPU用于等待I/O操作占用CPU总时间的百分比
* %steal:管理程序(hypervisor)为另一个虚拟进程提供服务而等待虚拟CPU的百分比
* %idle:CPU空闲时间百分比


1. 若 %iowait 的值过高，表示硬盘存在I/O瓶颈 
2. 若 %idle 的值高但系统响应慢时，有可能是CPU等待分配内存，此时应加大内存容量 
3. 若 %idle 的值持续低于1，则系统的CPU处理能力相对较低，表明系统中最需要解决的资源是 CPU

Device中各列参数含义如下：

* Device:设备名称
* tps:每秒向磁盘设备请求数据的次数，包括读、写请求，为rtps与wtps的和。出于效率考虑，每一次IO下发后并不是立即处理请求，而是将请求合并(merge)，这里tps指请求合并后的请求计数。
* Blk_read/s:Indicate the amount of data read from the device expressed in a number of blocks per second. Blocks are equivalent to sectors with kernels 2.4 and later and therefore have a size of 512 bytes. With older kernels, a block is of indeterminate size.
* Blk_wrtn/s	Indicate the amount of data written to the device expressed in a number of blocks per second.
* Blk_read:取样时间间隔内读扇区总数量
* Blk_wrtn:取样时间间隔内写扇区总数量


我们可以使用-c选项单独显示avg-cpu部分的结果，使用-d选项单独显示Device部分的信息。

# 指定采样时间间隔与采样次数

以”iostat interval [count] ”形式指定iostat命令的采样间隔和采样次数：

````
$ iostat -d 2 3
Linux 2.6.32-279.19.3.el6.ucloud.x86_64 (vm1)   06/12/2017  _x86_64_    (8 CPU)

Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
vda               0.45         0.29         8.10    6634946  183051408
vdb               0.12         3.11        30.55   70342034  689955328

Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
vda               0.00         0.00         0.00          0          0
vdb               0.00         0.00         0.00          0          0

Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
vda               1.50         0.00        12.00          0         24
vdb               0.00         0.00         0.00          0          0

````

以上命令输出Device的信息，采样时间为1秒，采样2次，若不指定采样次数，则iostat会一直输出采样信息，直到按”ctrl+c”退出命令。注意，第1次采样信息与单独执行iostat的效果一样，为从系统开机到当前执行时刻的统计信息。

# 以kB为单位显示读写信息(-k选项)/以mB为单位显示读写信息(-m选项)

我们可以使用-k选项，指定iostat的部分输出结果以kB为单位，而不是以块数为单位：

````
$ iostat -d -k
Linux 2.6.32-279.19.3.el6.ucloud.x86_64 (vm1)   06/12/2017  _x86_64_    (8 CPU)

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               0.45         0.15         4.05    3317473   91525980
vdb               0.12         1.56        15.27   35171017  344977664
````

以上输出中，kB_read/s、kB_wrtn/s、kB_read和kB_wrtn的值均以kB为单位，相比以块数为单位，这里的值为原值的一半(1kB=512bytes*2)

# 更详细的io统计信息(-x选项)

为显示更详细的io设备统计信息，我们可以使用-x选项，在分析io瓶颈时，一般都会开启-x选项：

````
$ iostat -x -k -d 1
Linux 2.6.32-279.19.16.el6.ucloud.x86_64 (yg-uhost724)  06/12/2017  _x86_64_    (24 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util
sda               0.00  9915.00    1.00   90.00     4.00 34360.00   755.25    11.79  120.57   6.33  57.60
````

以上各列的含义如下：

* rrqm/s:每秒对该设备的读请求被合并次数，文件系统会对读取同块(block)的请求进行合并
* wrqm/s:每秒对该设备的写请求被合并次数
* r/s:每秒完成的读次数
* w/s:每秒完成的写次数
* rkB/s:每秒读数据量(kB为单位)
* wkB/s:每秒写数据量(kB为单位)
* avgrq-sz:平均每次IO操作的数据量(扇区数为单位)
* avgqu-sz:平均等待处理的IO请求队列长度
* await:平均每次IO请求等待时间(包括等待时间和处理时间，毫秒为单位)
* svctm:平均每次IO请求的处理时间(毫秒为单位)
* %util:采用周期内用于IO操作的时间比率，即IO队列非空的时间比率

对于以上示例输出，我们可以获取到以下信息： 
- 每秒向磁盘上写30M左右数据(wkB/s值) 
- 每秒有91次IO操作(r/s+w/s)，其中以写操作为主体 
- 平均每次IO请求等待时间为120.57毫秒，处理时间为6.33毫秒 
- 等待处理的IO请求队列中，平均有11.79个请求驻留
