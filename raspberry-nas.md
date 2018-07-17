---
title: 利用树莓派搭建一个简易的NAS
date: 2017-07-20 22:39:35
tags: [raspberrypi,python,NAS]
categories: raspberrypi
---
# 准备
* raspberry pi 3
* 硬盘（格式化过ext4的）
* 连接raspberry用的终端

<!-- more -->

# 安装samba
````bash
sudo apt-get update
sudo apt-get install samba samba-common-bin
````

# 配置samba
````bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.back
sudo vim /etc/samba/smb.conf
````
````bash
# 在末尾加入如下内容
# 分享名称
[MyNAS]
# 说明信息
comment = NAS Storage
# 可以访问的用户
valid users = pi,root
# 共享文件的路径,raspberry pi 会自动将连接到其上的外接存储设备挂载到/media/pi/目录下。
path = /media/pi/
# 可被其他人看到资源名称（非内容）
browseable = yes
# 可写
writable = yes
# 新建文件的权限为 664
create mask = 0664
# 新建目录的权限为 775
directory mask = 0775
````
可以把配置文件中你不需要的分享名称删除，例如 [homes], [printers] 等。
测试配置文件是否有错误，根据提示做相应修改
````bash
testparm
````
添加登陆账户并创建密码，必须是 linux 已存在的用户
````bash
sudo smbpasswd -a pi
````
重启 samba 服务
````bash
sudo /etc/init.d/samba restart
````
# 连接
一般树莓派跟你的WiFi相连的话，你的网络就能看到跟上面配置一样的分享名称，如mac上面这样的显示：
[![](http://idiotsky.top/images/nas-screenshot.png)](http://idiotsky.top/images/nas-screenshot.png) 
如果显示没权限，可以断开连接，用你上面添加的账号登录。

# 问题总结
* 基本上不是ext4格式的硬盘都不用上传到NAS上了，因为树莓对其他格式的硬盘只有读权限。
* 如果是ext4格式，也不要高兴，那上传速度可以😭的
* 如果是其他格式的话，上面都说只能读，一般情况拷进硬盘的片片是可以用pi用户读的，如果遇到连pi用户都没有读权限的话，而且登上树莓派进到硬盘里面强制改权限都是改不了的。所以，老老实实加个root用户吧。
    ````bash
    sudo smbpasswd -a root
    ````
* 用root用户连上去基本没有不能读的。但是还是不能写。老老实实还是拔硬盘到电脑烤吧。