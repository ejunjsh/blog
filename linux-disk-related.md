---
title: Linux硬盘分区相关命令
date: 2014-02-30 17:23:33
tags: [fdisk,e2label,mkfs,mount,umount,df,du,linux]
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

# 查看文件夹大小-du
du的英文原义为“disk usage”，含义为显示磁盘空间的使用情况，统计目录（或文件）所占磁盘空间的大小。该命令的功能是逐级进入指定目录的每一个子目录并显示该目录占用文件系统数据块（1024字节）的情况。若没有给出指定目录，则对当前目录进行统计。

df命令的各个选项含义如下：
````
  -s：对每个Names参数只给出占用的数据块总数。
  -a：递归地显示指定目录中各文件及子目录中各文件占用的数据块数。若既不指定-s，也不指定-a，则只显示Names中的每一个目录及其中的各子目录所占的磁盘块数。
  -b：以字节为单位列出磁盘空间使用情况（系统默认以k字节为单位）。
  -k：以1024字节为单位列出磁盘空间使用情况。
  -c：最后再加上一个总计（系统默认设置）。
  -l：计算所有的文件大小，对硬链接文件，则计算多次。
  -x：跳过在不同文件系统上的目录不予统计。
  -h: 人类可读的显示出对应的文件夹的大小，例如（10K，22M）
````
下面举例说明du命令的使用：
````shell
# 查看/mnt目录占用磁盘空间的情况
$ du –abk /mnt
1       /mnt/cdrom
1       /mnt/floppy
3       /mnt
 
# 列出各目录所占的磁盘空间，但不详细列出每个文件所占的空间
# 输出清单中的第1列是以块为单位计的磁盘空间容量，第2列列出目录中使用这些空间的目录名称。
$ du
3684    ./log
84      ./libnids-1.17/doc
720     ./libnids-1.17/src
32      ./libnids-1.17/samples
1064    ./libnids-1.17
4944    .
````

这可能是一个很长的清单，有时只需要一个总数。这时可在du命令中加-s选项来取得总数：

````shell
$ du –s /mnt 
3       /mnt
 
# 列出所有文件和目录所占的空间（使用a选项），并以字节为单位（使用b选项）来计算大小
$ du –ab /root/mail
6144    mail/sent-mail
1024    mail/saved-messages
8192    mail
````

其实用-h 选项可以显示更加可读的数据
````shell
$ du -h
12K     ./.gnupg
0       ./bin
16K     ./.ssh
0       ./.local/share/systemd
0       ./.local/share
0       ./.local
8.0K    ./.vim/colors
8.0K    ./.vim
175M    .
````