---
title: openstack安装-keystone
date: 2017-09-09 15:46:36
tags: [openstack,ubuntu,kvm,keystone]
categories: openstack
---
> keystone 是openstack中所有service的权限管理和接口入口，所以先安装它
> 这一章都是在`controller`节点操作。。。

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

# 编写admin环境变量脚本
创建一个`admin-openrc`文件，内容如下：
````
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
````
**`ADMIN_PASS` 用上面的创建的密码替换掉这个**

# 创建域(domain),项目(project),用户(users)和角色(roles)
创建这些之前，先执行上面那个脚本，切换到admin用户。
````bash
. admin-openrc
````
因为OpenStack默认创建了`default`的域，所以这次不用创建域了
1. 创建一个`service` project
    ````bash
    $ openstack project create --domain default \
    --description "Service Project" service

    +-------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    | description | Service Project                  |
    | domain_id   | default                          |
    | enabled     | True                             |
    | id          | 24ac7f19cd944f4cba1d77469b2a73ed |
    | is_domain   | False                            |
    | name        | service                          |
    | parent_id   | default                          |
    +-------------+----------------------------------+
    ````
2. 创建`demo`project,user和role
    ````bash
    $ openstack project create --domain default \
    --description "Demo Project" demo

    +-------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    | description | Demo Project                     |
    | domain_id   | default                          |
    | enabled     | True                             |
    | id          | 231ad6e7ebba47d6a1e57e1cc07ae446 |
    | is_domain   | False                            |
    | name        | demo                             |
    | parent_id   | default                          |
    +-------------+----------------------------------+
    ````
    ````bash
    $ openstack user create --domain default \
    --password-prompt demo

    User Password:
    Repeat User Password:
    +---------------------+----------------------------------+
    | Field               | Value                            |
    +---------------------+----------------------------------+
    | domain_id           | default                          |
    | enabled             | True                             |
    | id                  | aeda23aa78f44e859900e22c24817832 |
    | name                | demo                             |
    | options             | {}                               |
    | password_expires_at | None                             |
    +---------------------+----------------------------------+
    ````
    ````bash
    $ openstack role create user

    +-----------+----------------------------------+
    | Field     | Value                            |
    +-----------+----------------------------------+
    | domain_id | None                             |
    | id        | 997ce8d05fc143ac97d83fdfb5998552 |
    | name      | user                             |
    +-----------+----------------------------------+
    ````
    ````bash
    $ openstack role add --project demo --user demo user
    ````
3. 创建demo用户的环境脚本
    `demo-openrc`
    ````
    export OS_PROJECT_DOMAIN_NAME=Default
    export OS_USER_DOMAIN_NAME=Default
    export OS_PROJECT_NAME=demo
    export OS_USERNAME=demo
    export OS_PASSWORD=DEMO_PASS
    export OS_AUTH_URL=http://controller:5000/v3
    export OS_IDENTITY_API_VERSION=3
    export OS_IMAGE_API_VERSION=2
    ````
    **`DEMO_PASS`**替换成上一步创建demo用户时保存的密码
    好了现在执行一下这个脚本就切换到demo用户了
    ````bash
    . demo-openrc
    ````

# 验证
切换到`admin`用户
````bash
. admin-openrc
````

````bash
$ openstack token issue

+------------+-----------------------------------------------------------------+
| Field      | Value                                                           |
+------------+-----------------------------------------------------------------+
| expires    | 2016-02-12T20:44:35.659723Z                                     |
| id         | gAAAAABWvjYj-Zjfg8WXFaQnUd1DMYTBVrKw4h3fIagi5NoEmh21U72SrRv2trl |
|            | JWFYhLi2_uPR31Igf6A8mH2Rw9kv_bxNo1jbLNPLGzW_u5FC7InFqx0yYtTwa1e |
|            | eq2b0f6-18KZyQhs7F3teAta143kJEWuNEYET-y7u29y0be1_64KYkM7E       |
| project_id | 343d245e850143a096806dfaefa9afdc                                |
| user_id    | ac3377633149401296f6c0d92d79dc16                                |
+------------+-----------------------------------------------------------------+
````
上面输出上面类似代表正常了。`demo`用户也是类似，这里就不演示了。

# 总结
这里需要注意的是，用户的环境脚本，其实他只是方便切换用户的，就算不做，都可以通过把环境变量作为`openstack`命令参数来执行，例如上面验证可以用下面命令：
````bash
$ openstack --os-auth-url http://controller:35357/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue

Password: #这里要输入admin密码
+------------+-----------------------------------------------------------------+
| Field      | Value                                                           |
+------------+-----------------------------------------------------------------+
| expires    | 2016-02-12T20:14:07.056119Z                                     |
| id         | gAAAAABWvi7_B8kKQD9wdXac8MoZiQldmjEO643d-e_j-XXq9AmIegIbA7UHGPv |
|            | atnN21qtOMjCFWX7BReJEQnVOAj3nclRQgAYRsfSU_MrsuWb4EDtnjU7HEpoBb4 |
|            | o6ozsA_NmFWEpLeKy0uNn_WeKbAhYygrsmQGA49dclHVnz-OMVLiyM9ws       |
| project_id | 343d245e850143a096806dfaefa9afdc                                |
| user_id    | ac3377633149401296f6c0d92d79dc16                                |
+------------+-----------------------------------------------------------------+
````

