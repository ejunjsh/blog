---
title: openstack安装准备(一)
date: 2017-08-18 22:50:07
tags: [openstack,ubuntu,kvm]
categories: openstack
---
# 准备VMware
由于我是习惯了mac上做实验，所以用VMware fusion，随便下个破解版即可。

# 准备Ubuntu
Ubuntu去官网下载16.04的服务器版本的ISO即可。

<!-- more -->

# 准备网络
这次实验用到两台虚拟机： controller,compute
## controller
````bash
$ cat /etc/hostname
controller

$ cat /etc/hosts
127.0.0.1	localhost
192.168.199.11  compute
192.168.199.10  controller

$ cat /etc/network/interfaces
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto ens33
iface ens33 inet static
address 192.168.199.10
netmask 255.255.255.0
gateway 192.168.199.1
dns-nameservers 192.168.199.1

auto ens34
iface ens34 inet static
address 10.170.56.10
netmask 255.255.255.0

auto ens35
iface ens35 inet manual
````

接口|ip|模式
----|----|-----
ens33|192.168.199.10| 桥接
ens34|10.170.56.10| 私有
ens35|网关192.168.112.2| nat

## compute
````bash
$ cat /etc/hostname
compute

$ cat /etc/hosts
127.0.0.1	localhost
192.168.199.11  compute
192.168.199.10  controller

$ cat /etc/network/interfaces
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto ens33
iface ens33 inet static
address 192.168.199.11
netmask 255.255.255.0
gateway 192.168.199.1
dns-nameservers 192.168.199.1

auto ens34
iface ens34 inet static
address 10.170.56.11
netmask 255.255.255.0
````

接口|ip|模式
----|----|-----
ens33|192.168.199.11| 桥接
ens34|10.170.56.11| 私有

# 总结
PS:
* 桥接模式是虚拟机可以更物理机所在网络共享一套网络，例如跟物理机同一个WiFi里面的设备都可以访问物理机里面的虚拟机。这里用来做管理节点的网络。
* 私有模式代表虚拟机只能跟物理机作为一个网络，其他设备访问不了，一般可以用来做内部网络
* nat模式用来给虚拟机访问互联网用

PSPS:
接下来会在上面的两台虚拟机安装openstack，安装完openstack后，两台虚拟机对于openstack来说，就是物理机，通过openstack，创建的就是云主机（或者叫租户）了。所以必须要谨记这点。

PSPSPS:
* 桥接模式的ip必须是你电脑所在网络的任意不冲突的同子网的ip
* 私有模式的ip可以任意一个子网下的ip，这个网络是用来做租户网络的
* nat网络不用配ip，这个给租户用来访问外网的，接下来实验会再提及，注意下他的网关即可，它是你的VMware的nat的一个网关。



上面网络配置好后，可以开搞了，至于怎么安装虚拟机和配置网络，可以搜索相关文章😈。
