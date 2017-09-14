---
title: centos 启用ftp功能
date: 2014-8-17 21:50:01
tags: linux
categories: linux
---
1.安装vsftpd组件
````bash
yum -y install vsftpd
````
<!-- more -->
安装完后，有个/etc/vsftpd/vsftpd.conf 文件，用来配置
还有自动新建了一个ftp用户和ftp的组，home目录为/var/ftp,默认是nologin（不能登录系统）
````bash
cat /etc/passwd | grep ftp
````
默认ftp服务是没有启动的，用下面命令启动
````bash
service vsftpd start
````
2.安装ftp客户端组件（用来验证是否vsftpd）
````bash
yum -y install ftp
````
执行命令尝试登录
````bash
ftp localhost
````
输入用户名ftp，密码随便（因为默认是允许匿名的）

登录成功，就代表ftp服务可用了。

但是，外网是访问不了的，所以还要继续配置。

3.取消匿名登陆
````bash
vi /etc/vsftpd/vsftpd.conf
````
把第一行的 anonymous_enable=YES ，改为NO
重启
````bash
service vsftpd restart
````
4.新建一个用户(ftpuser为用户名，随便就可以)
````bash
useradd ftpuser
````
修改密码（输入两次）
````bash
passwd ftpuser
````
这样一个用户建完，可以用这个登录，记得用普通登录不要用匿名了。登录后默认的路径为 /home/ftpuser.

5.开放21端口

因为ftp默认的端口为21，而centos默认是没有开启的，所以要修改iptables文件
````bash
vi /etc/sysconfig/iptables
````
在行上面有22 -j ACCEPT 下面另起一行输入跟那行差不多的，只是把22换成21，然后：wq保存。
还要运行下,重启iptables
````bash
service iptables restart
````
外网是可以访问上去了，可是发现没法返回目录，也上传不了，因为selinux作怪了。
6.修改selinux
````bash
getsebool -a | grep ftp
````
执行上面命令，再返回的结果看到两行都是off，代表，没有开启外网的访问
````bash
.... 
allow_ftpd_full_access off 
....
....
ftp_home_dir off
````
只要把上面都变成on就行
执行
````bash
setsebool -P allow_ftpd_full_access 1 
setsebool -P ftp_home_dir off 1
````
再重启一下vsftpd
````bash
service vsftpd restart
````
这样应该没问题了（如果，还是不行，看看是不是用了ftp客户端工具用了passive模式访问了，如提示Entering Passive mode，就代表是passive模式，默认是不行的，因为ftp passive模式被iptables挡住了，下面会讲怎么开启，如果懒得开的话，就看看你客户端ftp是否有port模式的选项，或者把passive模式的选项去掉。如果客户端还是不行，看看客户端上的主机的电脑是否开了防火墙，关吧）

7.开启passive模式

默认是开启的，但是要指定一个端口范围，打开vsftpd.conf文件，在后面加上
````bash
pasv_min_port=30000

pasv_max_port=30999
````

表示端口范围为30000~30999，这个可以随意改。
改完重启一下vsftpd

由于指定这段端口范围，iptables也要相应的开启这个范围，所以像上面那样打开iptables文件

也是在21上下面另起一行，更那行差不多，只是把21 改为30000:30999,然后:wq保存，重启下iptables。这样就搞定了。
