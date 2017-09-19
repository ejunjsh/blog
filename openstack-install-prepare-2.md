---
title: openstack安装准备(二)-环境的安装
date: 2017-08-19 22:50:07
tags: [openstack,ubuntu,kvm]
categories: openstack
---
[![](http://idiotsky.me/images/openstack-install-prepare-2-1.png)](http://idiotsky.me/images/openstack-install-prepare-2-1.png)
上图是整个openstack的架构图，里面的椭圆方块都是openstack的服务，所以安装openstack就是要安装这些服务。

按照官方建议，这次openstack安装的服务为：
* Identity service (keystone)
* Image service (glance)
* Compute service (nova)
* Networking service (neutron)
* Dashboard (horizon)
* Block Storage service (cinder)

<!-- more -->
在安装上面服务前，先要弄好环境的😁
这次安装的openstack为最新的release，**pike**
# 安装openstack仓库
````bash
# change to root
$ sudo -i
$ apt install software-properties-common
$ add-apt-repository cloud-archive:pike
$ apt update && apt dist-upgrade
$ apt install python-openstackclient
````
上面的步骤两个节点都要安装。
**以下步骤安装在controller**
# 安装数据库
openstack所用到的数据都会存到数据库里，所以安装一个数据库是准备的一个重要步骤。mariadb是官方建议的数据库。
## 安装和配置mariadb
````bash
$ apt install mariadb-server python-pymysql
$ vi /etc/mysql/mariadb.conf.d/99-openstack.cnf
# 加一个[mysqld]区，bind-address为管理网络ip
[mysqld]
bind-address = controller # 192.168.199.10 

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
````
## 收尾
重启服务
````bash
$ service mysql restart
````
设置下root用户的密码，这个密码后面要用到，务必谨记。
````bash
$ mysql_secure_installation
````

# 安装消息队列
openstack用消息队列来异步控制各种service，所以要装一个，rabbitmq是官方推荐，装之。
## 安装和配置rabbitmq
````bash
$ apt install rabbitmq-server
````
加一个openstack用户。
````bash
$ rabbitmqctl add_user openstack RABBIT_PASS #用你的密码替换下RABBIT_PASS，谨记这个密码，后面有用。
````
赋予更多权限给openstack用户
````bash
$ rabbitmqctl set_permissions openstack ".*" ".*" ".*" 
````
# 安装缓存
openstack用到缓存，memcached是官方推荐，还是装之。
## 安装和配置memcached
````bash
$ apt install memcached python-memcache
$ vi /etc/memcached.conf
#监听管理网络ip
#-l 127.0.0.1 改成下面这样
-l controller # 192.168.199.10 
````
## 收尾
重启服务
````bash
$ service memcached restart
````

# 总结
基本上环境已经搭好了，接下来就要安装各种服务了。😈