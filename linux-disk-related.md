---
title: Linux硬盘分区相关命令
date: 2014-02-30 17:23:33
tags: [fdisk,e2label,mkfs,mount,umount,df]
categories: linux命令
---
# 分区fdisk
## fdisk命令参数介绍
`fdisk -l` 显示所有分区详细信息
`fdisk [设备名]`进入交互模式，以下是交互参数：
* p、打印分区表。
* n、新建一个新分区。
* d、删除一个分区。
* q、退出不保存。
* w、把分区写进分区表，保存并退出。

<!-- more -->
## 实例
````bash
$ fdisk /dev/hdd   
````
[![](http://idiotsky.me/images/linux-disk-related-1.jpg)](http://idiotsky.me/images/linux-disk-related-1.jpg)
按"p"键打印分区表
[![](http://idiotsky.me/images/linux-disk-related-2.jpg)](http://idiotsky.me/images/linux-disk-related-2.jpg)
这块硬磁尚未分区
按"n"键新建一个分区。
[![](http://idiotsky.me/images/linux-disk-related-3.jpg)](http://idiotsky.me/images/linux-disk-related-3.jpg)
出现两个菜单e表示扩展分区，p表示主分区
按"p"键出现提示："Partition number (1-4): "选择主分区号
输入"1"表示第一个主分区。
[![](http://idiotsky.me/images/linux-disk-related-4.jpg)](http://idiotsky.me/images/linux-disk-related-4.jpg)
直接按回车表示1柱面开始分区。
[![](http://idiotsky.me/images/linux-disk-related-5.jpg)](http://idiotsky.me/images/linux-disk-related-5.jpg)
提示最后一个柱面或大小。
输入+5620M 按回车
表示第一个分区为5G空间。
按"p"查看一下分区
[![](http://idiotsky.me/images/linux-disk-related-6.jpg)](http://idiotsky.me/images/linux-disk-related-6.jpg)
这样一个主分区就分好了。
接下来分第二个主分区，把剩余空间都给第二个主分区。
按"n"
键新增一个分区
按"p"键设为主分区
输入"2"把主分区编号设为2
按两下回车把剩余空间分给第二个主分区。
按"p"键打印分区表
[![](http://idiotsky.me/images/linux-disk-related-7.jpg)](http://idiotsky.me/images/linux-disk-related-7.jpg)
按"w"键保存退出。 
读者可根据自己的硬盘大小来划分合适的分区。

# 格式化mkfs
分好区要使用的话，还要格式化下`mkfs -t ext4 [分区名]`
````bash
$ mkfs -t ext4 /dev/hdd1
$ mkfs -t ext4 /dev/hdd2
````

# 挂载分区mount
````bash
$ mount /dev/hdd1 /hdd1
$ mount /dev/hdd2 /hdd2
````

# 查看挂载df
````bash
$ df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/hda3             7.5G  2.8G  4.3G  40% /
/dev/hda1              99M   17M   78M  18% /boot
tmpfs                  62M     0   62M   0% /dev/shm
/dev/hdd1             2.5G   68M  2.3G   3% /hdd1
/dev/hdd2             2.5G   68M  2.3G   3% /hdd2
````
上面的两个分区分别挂载在`/hdd1`,`/hdd2`

# 修改标签e2label
一般上面的分区之后都分配一个很长的label，可以通过`e2label [分区] [新label]` 改下标签
````bash
$ e2label /dev/hdd1 mydisk
````

# 卸载分区umount
命令类似于`mount`
````bash
$ umount /dev/hdd1 /hdd1
$ umount /dev/hdd2 /hdd2
````