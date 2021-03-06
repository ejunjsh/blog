---
title: openstack安装-nova(一)
date: 2017-09-11 19:11:12
tags: [openstack,ubuntu,kvm,nova]
categories: openstack
---
> nova是OpenStack一个核心服务，提供计算服务，主要负责虚拟机的各种操作，如启动，销毁，快照，还有选择合适的compute节点部署虚拟机。
> nova中的服务有controller和compute之分，是一对多的关系，即一个controller可以有多个compute，所以一些controller服务要装到`controller`节点上，compute服务可以装到`compute`节点，也可以装到`controller`节点来让它充当一部分compute的能力。
> 由于nova涉及`controller`节点和`compute`节点的安装，所以分了两篇文章来讲解
> 这一篇先介绍怎么安装nova的controller服务在`controller`节点上。

# 准备
## 创建数据库
* 用`root`用户权限执行`mysql`
````shell
$ mysql
````
* 创建`nova_api`, `nova`,和`nova_cell0`数据库：
````sql
MariaDB [(none)]> CREATE DATABASE nova_api;
MariaDB [(none)]> CREATE DATABASE nova;
MariaDB [(none)]> CREATE DATABASE nova_cell0;
````
* 赋予适合的权限给这些数据库
````sql
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';
````
替换`NOVA_DBPASS`密码为合适的，下面会用到。
* 退出mysql

<!-- more -->
## 执行以下命令，进入admin身份
````shell
$ . admin-openrc
````
## 创建计算服务相关权限用户
* 创建`nova`用户：
````shell
$ openstack user create --domain default --password-prompt nova

User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 8a7dbf5279404537b1c7b86c033620fe |
| name                | nova                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
````
这里的密码要记住，下面会用到。
* 给`nova`用户添加`admin`角色
````shell
$ openstack role add --project service --user nova admin
````
* 创建`nova`服务
````shell
$ openstack service create --name nova \
  --description "OpenStack Compute" compute

+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | 060d59eac51b4594815603d75a00aba2 |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
````
## 创建计算服务API endpoints
````shell
$ openstack endpoint create --region RegionOne \
  compute public http://controller:8774/v2.1

+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 3c1caa473bfe4390a11e7177894bcc7b          |
| interface    | public                                    |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | 060d59eac51b4594815603d75a00aba2          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1               |
+--------------+-------------------------------------------+

$ openstack endpoint create --region RegionOne \
  compute internal http://controller:8774/v2.1

+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | e3c918de680746a586eac1f2d9bc10ab          |
| interface    | internal                                  |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | 060d59eac51b4594815603d75a00aba2          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1               |
+--------------+-------------------------------------------+

$ openstack endpoint create --region RegionOne \
  compute admin http://controller:8774/v2.1

+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 38f7af91666a47cfb97b4dc790b94424          |
| interface    | admin                                     |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | 060d59eac51b4594815603d75a00aba2          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1               |
+--------------+-------------------------------------------+
````
## 创建一个布置服务（Placement service）的用户
````shell
$ openstack user create --domain default --password-prompt placement

User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | fa742015a6494a949f67629884fc7ec8 |
| name                | placement                        |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
````
这里的密码也要记住，下面会用到。
## 给这个用户添加admin权限
````shell
$ openstack role add --project service --user placement admin
````
## 创建布置服务的api入口
````shell
$ openstack service create --name placement --description "Placement API" placement
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Placement API                    |
| enabled     | True                             |
| id          | 2d1a27022e6e4185b86adac4444c495f |
| name        | placement                        |
| type        | placement                        |
+-------------+----------------------------------+
````
## 创建布置服务api endpoints
````shell
$ openstack endpoint create --region RegionOne placement public http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 2b1b2637908b4137a9c2e0470487cbc0 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2d1a27022e6e4185b86adac4444c495f |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne placement internal http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 02bcda9a150a4bd7993ff4879df971ab |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2d1a27022e6e4185b86adac4444c495f |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne placement admin http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 3d71177b9e0f406f98cbff198d74b182 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2d1a27022e6e4185b86adac4444c495f |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
````

# 安装和配置组件
## 安装包
````shell
$ apt install nova-api nova-conductor nova-consoleauth \
  nova-novncproxy nova-scheduler nova-placement-api
````
## 修改`/etc/nova/nova.conf`
* 修改`[api_database]`和`[database]`区域
````ini
[api_database]
# ...
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api

[database]
# ...
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova
````
替换`NOVA_DBPASS`为你上面配置数据库的密码
* 在`[Default]`区域，对`RabbitMQ`配置
````ini
[DEFAULT]
# ...
transport_url = rabbit://openstack:RABBIT_PASS@controller
````
替换`RABBIT_PASS`为当时安装rabbitmq时候设置的密码
* 在`[api]`和`[keystone——authtoken]`区域，配置keystone相关配置
````ini
[api]
# ...
auth_strategy = keystone

[keystone_authtoken]
# ...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = NOVA_PASS
````
替换`NOVA_PASS`为你创建nova用户时的密码
* 在`[DEFAULT]`区域，配置`my_ip`为管理网络ip，此为controller节点的管理ip
````ini
[DEFAULT]
# ...
my_ip = 192.168.199.10
````
* 在`[DEFAUL]`区域，启用neutron作为网络服务的组件
````ini
[DEFAULT]
# ...
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
````
* 配置vnc
````ini
[vnc]
enabled = true
# ...
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip
````
* 在`[glance]`区域，配置glance
````ini
[glance]
# ...
api_servers = http://controller:9292
````
* 在`[oslo_concurrency]`区域，配置锁定路径
````ini
[oslo_concurrency]
# ...
lock_path = /var/lib/nova/tmp
````
* 在`[DEFAULT]`删除`log_dir`选项
* 在`[placement]`区域配置placement api
````ini
[placement]
# ...
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:35357/v3
username = placement
password = PLACEMENT_PASS
````
使用创建placement用户时的密码替换`PLACEMENT_PASS`

## 初始化`nova-api`数据库
````shell
$ su -s /bin/sh -c "nova-manage api_db sync" nova
````

## 注册`cell0`数据库
````shell
$ su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
````

## 创建`cell`
````shell
# su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
109e1d4b-536a-40d0-83c6-5f121b82b650
````

## 初始化`nova`数据库
````shell
$ su -s /bin/sh -c "nova-manage db sync" nova
````

## 验证`nova`的`cell0`和`cell1`
````shell
$ nova-manage cell_v2 list_cells
+-------+--------------------------------------+
| Name  | UUID                                 |
+-------+--------------------------------------+
| cell1 | 109e1d4b-536a-40d0-83c6-5f121b82b650 |
| cell0 | 00000000-0000-0000-0000-000000000000 |
+-------+--------------------------------------+
````

# 重启服务
````shell
$ service nova-api restart
$ service nova-consoleauth restart
$ service nova-scheduler restart
$ service nova-conductor restart
$ service nova-novncproxy restart
````