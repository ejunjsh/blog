---
title: openstack安装-keystone
date: 2017-09-09 15:46:36
tags: [openstack,ubuntu,kvm]
categories: openstack
---
> keystone 是openstack中所有service的权限管理和接口入口，所以先安装它

# 前提
1. 切换到`root`用户，执行下面命令
    ````bash
    $ mysql
    ````
2. 创建`keystone`数据库:
    ````bash
    MariaDB [(none)]> CREATE DATABASE keystone;
    ````
3. 赋予合适权限给`keystone`数据库：
    ````bash
    MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
    IDENTIFIED BY 'KEYSTONE_DBPASS';
    MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
    IDENTIFIED BY 'KEYSTONE_DBPASS';
    ````
    **用一个合适的密码覆盖`KEYSTONE_DBPASS`**
4. 退出数据库

<!-- more -->
# 安装和配置组件
1. 运行下面命令安装包：
    ````bash
    $ apt install keystone  apache2 libapache2-mod-wsgi
    ````
2. 编辑`/etc/keystone/keystone.conf`
    * 在`[database]` 区域，配置数据库访问连接：
        ````
        [database]
        # ...
        connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
        ````
        **替换掉`KEYSTONE_DBPASS`,密码是上面配置的数据库密码**
        **去掉这个区域里面其他关于`connection`的属性**
    * 在`[token]` 区域，配置Fernet：
        ````
        [token]
        # ...
        provider = fernet
        ````
3. 部署身份服务（Identity service）数据库：
    ````bash
    $ su -s /bin/sh -c "keystone-manage db_sync" keystone
    ````
4. 初始化Fernet
    ````bash
    $ keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
    $ keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
    ````
5. 启动身份服务（Identity service）
    ````bash
    $ keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
    --bootstrap-admin-url http://controller:35357/v3/ \
    --bootstrap-internal-url http://controller:5000/v3/ \
    --bootstrap-public-url http://controller:5000/v3/ \
    --bootstrap-region-id RegionOne
    ````
    **替换`ADMIN_PASS`**

# 配置Apache
因为这个服务是跑在Apache里面的，所以需要配置之。
1. 修改`/etc/apache2/apache2.conf`文件，配置`ServerName`选项：
    ````
    ServerName controller
    ````
2. 重启
    ````bash
    $ service apache2 restart
    ````

# 编写环境变量脚本
> to be continue...
