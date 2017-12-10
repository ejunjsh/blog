---
title: openstack安装-nova(二)
date: 2017-09-12 19:11:12
tags: [openstack,ubuntu,kvm,nova]
categories: openstack
---
> 现在要开始安装compute服务了，前面一章都说了，虽然controller节点主要用来做控制服务节点的，如果做实验的话，可以用它的多余资源来充当一下compute节点，那么这个实验就可以有两台compute节点了。
> 开始安装吧。。。

# 安装和配置组件
## 安装包
````shell
$ apt install nova-compute
````

## 修改`/etc/nova/nova.conf`
* 在`[DEFAULT]`区域，配置`RabbitMQ`
````ini
[DEFAULT]
# ...
transport_url = rabbit://openstack:RABBIT_PASS@controller
````
替换`RABBIT_PASS`为当时为`RabbitMQ`创建`openstack`用户时指定的密码
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
* 在`[DEFAULT]`区域，配置`my_ip`
````ini
[DEFAULT]
# ...
my_ip = 192.168.199.11
````
* 在`[DEFAULT]`区域，关掉防火墙和使用neutron
````ini
[DEFAULT]
# ...
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
````
以前nova是提供网络服务的，现在有了neutron之后，就不用了，所以这里要关掉先。
* 在`[vnc]`区域，配置远程访问：
````ini
[vnc]
# ...
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html
````
* 在`[glance]`区域，配置镜像访问接口：
````ini
[glance]
# ...
api_servers = http://controller:9292
````
* 在`[DEFAULT]`区域,`log_dir`选项
* 在`[placement]`区域，配置Placement API:
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
替换`PLACEMENT_PASS`为之前配置placement用户时候设置的密码

# 完成安装
## 配置虚拟机类型
执行以下命令：
````shell
$ egrep -c '(vmx|svm)' /proc/cpuinfo
````
如果返回多个，就代表你机子已经硬件支持kvm，所以不用配置什么
如果返回0，代表你机子不支持kvm，所以就需要修改为qemu
````ini
[libvirt]
# ...
virt_type = qemu
````

## 重启服务
````shell
$ service nova-compute restart
````

# 添加compute节点到cell数据库
> 以下操作只能在controller节点执行

1. 执行以下命令：
````shell
$ . admin-openrc

$ openstack compute service list --service nova-compute
+----+-------+--------------+------+-------+---------+----------------------------+
| ID | Host  | Binary       | Zone | State | Status  | Updated At                 |
+----+-------+--------------+------+-------+---------+----------------------------+
| 1  | node1 | nova-compute | nova | up    | enabled | 2017-04-14T15:30:44.000000 |
+----+-------+--------------+------+-------+---------+----------------------------+
````
2. 发现compute节点
````shell
# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova

Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting compute nodes from cell 'cell1': ad5a5985-a719-4567-98d8-8d148aaae4bc
Found 1 computes in cell: ad5a5985-a719-4567-98d8-8d148aaae4bc
Checking host mapping for compute host 'compute': fe58ddc1-1d65-4f87-9456-bc040dc106b3
Creating host mapping for compute host 'compute': fe58ddc1-1d65-4f87-9456-bc040dc106b3
````
如果你新加了个compute，必须要再执行一遍上面的命令，或者在nova的配置文件改一个合适的间隔来自动发现吧。
````ini
[scheduler]
discover_hosts_in_cells_interval = 300
````

# 总结
nova服务到这里已经都装完了，如果你controller也安装了compute的服务的话，那就有两台compute node了😄
接下来要进入重头戏的网络服务neutron的安装了👿