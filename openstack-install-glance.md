---
title: openstack安装-glance
date: 2017-09-10 14:22:32
tags: [openstack,ubuntu,kvm,glance]
categories: openstack
---
> glance提供镜像服务使用户能够发现，注册和检索虚拟机镜像。它提供了一个 REST API，可让您查询虚拟机镜像元数据并检索实际镜像。您可以存储镜像在各种位置（从简单的文件系统到像OpenStack Object Storage这样的对象存储系统）。
>为了简单起见，这里只是教你如何配置镜像服务，并把镜像保存到controller节点的文件目录里面，此目录为/var/lib/glance/images/。
> 这一章还是在`controller`节点操作。。。

# 准备
1. 创建数据库
* 用`root`用户权限执行`mysql`
````shell
$ mysql
````
* 创建`glance`数据库
````
MariaDB [(none)]> CREATE DATABASE glance;
````
* 赋予适合的权限给`glance`数据库
````
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
  IDENTIFIED BY 'GLANCE_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
  IDENTIFIED BY 'GLANCE_DBPASS';
````
替换一下`GLANCE_DBPASS`为一个合适的密码，后面会用到
* 退出mysql

2. 执行以下命令，进入admin身份
````shell
$ . admin-openrc
````

3. 创建服务
* 创建`glance`用户
````shell
$ openstack user create --domain default --password-prompt glance

User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 3f4e777c4062483ab8d9edd7dff829df |
| name                | glance                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
````
这里会提示输入密码，记住这个密码，下面会用到。
* 加`admin`角色到`glance`用户和`service`项目
````shell
$ openstack role add --project service --user glance admin
````
* 创建glance服务实体
````shell
$ openstack service create --name glance \
  --description "OpenStack Image" image

+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Image                  |
| enabled     | True                             |
| id          | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+
````

4. 创建镜像服务API endpoints
````shell
$ openstack endpoint create --region RegionOne \
  image public http://controller:9292

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 340be3625e9b4239a6415d034e98aace |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne \
  image internal http://controller:9292

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | a6e4b153c2ae4c919eccfdbb7dceb5d2 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne \
  image admin http://controller:9292

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 0c37ed58103f4300a84ff125a539032d |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
````

# 安装和配置组件
1. 安装包
````shell
$ apt install glance
````
2. 修改`/etc/glance/glance-api.conf`
* 在`[database]`区域，添加数据访问配置
````ini
[database]
# ...
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
````
替换`GLANCE_DBPASS`为前一节创建数据库的那个密码
* 在`[keystone_authtoken]`和`[paste_deploy]`区域，配置keystone
````ini
[keystone_authtoken]
# ...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = GLANCE_PASS

[paste_deploy]
# ...
flavor = keystone
````
替换`GLANCE_PASS `为之前你创建glance用户时的密码。
* 在`[glance_store]`区域，配置镜像存储路径。
````ini
[glance_store]
# ...
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
````
3. 修改`/etc/glance/glance-registry.conf`
* 在`[database]`区域，添加数据访问配置
````ini
[database]
# ...
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
````
替换`GLANCE_DBPASS`为前一节创建数据库的那个密码
* 在`[keystone_authtoken]`和`[paste_deploy]`区域，配置keystone
````ini
[keystone_authtoken]
# ...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = GLANCE_PASS

[paste_deploy]
# ...
flavor = keystone
````
替换`GLANCE_PASS `为之前你创建glance用户时的密码。
4. 初始化镜像服务数据库：
````
$ su -s /bin/sh -c "glance-manage db_sync" glance
````

# 重启服务
````shell
$ service glance-registry restart
$ service glance-api restart
````

# 验证
1. 进入`admin`身份
````shell
$ . admin-openrc
````
2. 下载测试镜像
````shell
$ wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
````
3. 上传测试镜像到镜像服务
````shell
$ openstack image create "cirros" \
  --file cirros-0.3.5-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public

+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | 133eae9fb1c98f45894a4e60d8736619                     |
| container_format | bare                                                 |
| created_at       | 2015-03-26T16:52:10Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/cc5c6982-4910-471e-b864-1098015901b5/file |
| id               | cc5c6982-4910-471e-b864-1098015901b5                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | cirros                                               |
| owner            | ae7a98326b9c455588edd2656d723b9d                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 13200896                                             |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2015-03-26T16:52:10Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+
````
4. 确认是否上传成功
````shell
$ openstack image list

+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 38047887-61a7-41ea-9b49-27987d5e8bb9 | cirros | active |
+--------------------------------------+--------+--------+
````