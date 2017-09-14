---
title: lvs+keepalived 做负载和热备
date: 2014-10-07 20:08:15
tags: [linux,lvs]
categories: linux
---
> 一直觉得nginx做负载不错了，可发现居然这玩意这么好，特别keepalived热备。

<!-- more -->
# 准备工作
本实验基于VMware+centos,以下是环境信息：
假如有4台服务器
master：192.168.153.132
backup：192.168.153.133
vip（virtual ip 虚拟ip）：192.168.153.100 （实际浏览器访问的地址）
realserver 1: 192.168.153.128
realserver 2: 192.168.153.129
热备的意思就是master坏了，backup能补上去，由于都是使用vip，所以外面浏览器访问没有影响
realserver1和2是用来做负载均衡，按照权重来调到相应的服务器来处理请求。
lvs和keepalived都要装到master和backup上

## 下载keepalived和lvs的admin程序
````bash
$ wget http://www.keepalived.org/software/keepalived-1.2.13.tar.gz
````
````bash
$ wget http://www.linuxvirtualserver.org/software/kernel-2.6/ipvsadm-1.26.tar.gz
````
## 安装依赖
````bash
$ yum -y install libnl* openssl* popt* kernel-devel
````
## 解压安装
````bash
$ tar -zxvf keepalived-1.2.13.tar.gz
$ tar -zxvf ipvsadm-1.26.tar.gz
````
````bash
$ cd keepalived-1.2.13
$ ./configure  --prefix=/usr  --sysconf=/etc
$ make && make install
````
````bash
$ cd ipvsadm-1.26
$ make && make install
````
## 验证lvs
````bash
$ ipvsadm ln
````
## 验证keepalived
````bash
$ service keepalived start
````
如果显示无误就代表安装成功了。

# 配置

## 配置master
````bash
$ vi /etc/keepalived/keepalived.conf
````
````
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}


vrrp_instance VI_1 {
    state MASTER   #表示master
    interface eth1 #网络接口，有可能是eth0，可以用命令ifconfig查看
    virtual_router_id 51
    priority 100   #优先级
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.153.100  #vip配置
    }
}


virtual_server 192.168.153.100 80 {
        delay_loop 6
        lb_algo rr   #负载算法
        lb_kind DR  #负载方式 DR，这种方式的话VIP端口和realserver端口必须一致
        #persistence_timeout 20
        protocol TCP
        real_server 192.168.153.128 80 {
        weight 3 #权重
        TCP_CHECK {
        connect_timeout 3
        nb_get_retry 3
        delay_before_retry 3
        connect_port 80
}
}
       real_server 192.168.153.129 80 {
        weight 3 #权重
        TCP_CHECK {
        connect_timeout 3
        nb_get_retry 3
        delay_before_retry 3
        connect_port 80
}
}

}
````
## 配置backup 
基本跟master一致，只是在上面红色的第一处改成BACKUP，第二处改成比master小的值，例如90

## 重启keepalived
````bash
$ service keepalived restart
````

## 配置realserver
在real server 1和2上，把以下脚本放在/etc/rc.d/init.d/下
````bash
#!/bin/bash
# description: Config realserver lo and apply noarp
 
SNS_VIP=192.168.153.100
 
. /etc/rc.d/init.d/functions
 
case "$1" in
start)
       ifconfig lo:0 $SNS_VIP netmask 255.255.255.255 broadcast $SNS_VIP
       /sbin/route add -host $SNS_VIP dev lo:0
       echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
       echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
       echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
       echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
       sysctl -p >/dev/null 2>&1
       echo "RealServer Start OK"
 
       ;;
stop)
       ifconfig lo:0 down
       route del $SNS_VIP >/dev/null 2>&1
       echo "0" >/proc/sys/net/ipv4/conf/lo/arp_ignore
       echo "0" >/proc/sys/net/ipv4/conf/lo/arp_announce
       echo "0" >/proc/sys/net/ipv4/conf/all/arp_ignore
       echo "0" >/proc/sys/net/ipv4/conf/all/arp_announce
       echo "RealServer Stoped"
       ;;
*)
       echo "Usage: $0 {start|stop}"
       exit 1
esac
 
exit 0
````
命名成realserver,然后可以用以下命令启动：
````bash
$ service realserver start
````

## 关闭iptable
如果不想关掉的话
在master 加上以下规则
````
-A INPUT -i eth1 -p vrrp -s 192.168.153.133 -j ACCEPT
````
在backup加上以下规则
````
-A INPUT -i eth1 -p vrrp -s 192.168.153.132 -j ACCEPT
````
其实就是让热备之间可以通讯，注意eth1是你的网络接口，如果是eth0就要换成eth0
realserver就只要开启80端口就可以了。

# 验证
* 开启realserver的web应用程序
* 执行realserver脚本
* 开启master和backup的keepalived
* 用`watch ipvsadm -ln`在master和backup上可以查看状态
如果有两条记录分别指向realserver的ip就代表运行正确了。
* 浏览器访问`vip`，看能否正确访问web应用程序。
* 关掉master 的keepalived，如果还能访问的话，就代表backup接管了。
可以在backup的日志上面看到，`tail -f /var/log/message`
如果有`transition MASTER`的，就代表接管了。
* 再启动master的keepalived，再到backup的日志上面看到`transition BACKUP`的，就代表已经交回master了。
* 关掉其中一个realserver，用`watch ipvsadm -ln`可以看到该realserver的ip被剔除了。

