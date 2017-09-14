---
title: SSH免密码登陆以及穿越跳板机
date: 2014-03-27 17:54:41
tags: [ssh,linux]
categories: linux命令
---
# 免密码直连  [user@hostA ~] $ssh hostB
## STEP1. 在hostA上生成RSA公钥私钥（在~/.ssh/下生成RSA私钥id\_rsa，公钥id_rsa.pub）
````bash
[user@hostA ~]$ ssh-keygen -t rsa
````
## STEP2. 将hostA的公钥传给hostB机器（在hostB机器的.ssh/authorized\_keys里面添加A机器的id_rsa.pub）
方法一：使用 ssh-copy-id工具
````bash
[user@hostA ~]$ ssh-copy-id -i .ssh/id_rsa.pub hostB
````
方法二：人肉拼接
````bash
[user@hostA ~]$ scp .ssh/id_rsa.pub hostB:~/.ssh/temp
[user@hostB ~]$ cat .ssh/temp >> .ssh/authorized_keys
````
<!-- more -->

# 穿越跳板机 desktop -> gateway -> server
在我厂，从desktop访问server，必须先登录到gateway，即:
````bash
[user@desktop ~]$ ssh gateway
[user@gateway ~]$ ssh server
````
根据"1. 免密码直连”虽然可以减少两次输入密码，但仍然很麻烦。如果可以一步到位该多好。另外，SCP文件更郁闷，先把文件从desktop拷到gateway，再拷到server，奔溃了。。
## STEP1. 在desktop上生成RSA公钥私钥，方法同1.STEP1
## STEP2. 在desktop的.ssh目录下生成config文件，文件内容如下
````
Host shortServer
    ProxyCommand ssh user@gateway nc server %p 2>/dev/null
````
其中，gateway为跳板机ip，server为服务器ip，shortServer自定义一个缩写的服务器名称。
例如：我厂gateway地址x.y.g.w，以及N多服务器，例如*.a.b.c。
````
Host 123
    ProxyCommand ssh user@x.y.g.w nc 123.a.b.c %p 2>/dev/null
````
## STEP3. 把deskptop的公钥分发给gateway和server
分发给gateway，因为可以直连desktop和gateway，直接用1.STEP2.方法一
分发给server，因为暂时无法直连desktop和server，所以只能使用1.STEP2.方法二

## 连接方法： 
````bash
[user@desktop ~]$ ssh 123
````
此时SCP也能直接用了
````bash
[user@desktop ~]$ scp ToCopy.txt 123:
````