---
title: Linux netstat命令详解
date: 2018-12-02 22:04:11
tags: [linux,netstat]
categories: linux命令
---

> mark and index for searching 👿

# 简介

Netstat 命令用于显示各种网络相关信息，如网络连接，路由表，接口状态 (Interface Statistics)，masquerade 连接，多播成员 (Multicast Memberships) 等等。

# 输出信息含义

执行netstat后，其输出结果为

````shell
# netstat|more
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State
tcp        0      0 10.0.2.15:ssh               10.0.2.2:52091              ESTABLISHED
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node Path
unix  11     [ ]         DGRAM                    11021  /dev/log
unix  2      [ ]         DGRAM                    11556  @/org/freedesktop/hal/udev_event
unix  2      [ ]         DGRAM                    8986   @/org/kernel/udev/udevd
unix  3      [ ]         STREAM     CONNECTED     17777
unix  3      [ ]         STREAM     CONNECTED     17776
````

<!-- more -->

从整体上看，netstat的输出结果可以分为两个部分：

一个是Active Internet connections，称为有源TCP连接，其中"Recv-Q"和"Send-Q"指的是接收队列和发送队列。这些数字一般都应该是0。如果不是则表示包正在队列中堆积。这种情况只能在服务器网络繁忙的情况见到。

````
   Active Internet connections (TCP, UDP, raw)
   Proto
       The protocol (tcp, udp, raw) used by the socket.

   Recv-Q
       The count of bytes not copied by the user program connected to this socket.

   Send-Q
       The count of bytes not acknowledged by the remote host.
````

另一个是Active UNIX domain sockets，称为有源Unix域套接口(和网络套接字一样，但是只能用于本机通信，性能可以提高一倍)。

* Proto显示连接使用的协议，
* RefCnt表示连接到本套接口上的进程数量，
* Types显示套接口的类型，
* State显示套接口当前的状态，
* Path表示连接到套接口的其它进程使用的路径名。

````
 Active UNIX domain Sockets
   Proto
       The protocol (usually unix) used by the socket.

   RefCnt
       The reference count (i.e. attached processes via this socket).

   Flags
       The  flags  displayed  is  SO_ACCEPTON  (displayed as ACC), SO_WAITDATA (W) or SO_NOSPACE (N).  SO_ACCECPTON is used on
       unconnected sockets if their corresponding processes are waiting for a connect request. The other flags are not of nor-
       mal interest.

   Type
       There are several types of socket access:

       SOCK_DGRAM
              The socket is used in Datagram (connectionless) mode.

       SOCK_STREAM
              This is a stream (connection) socket.

       SOCK_RAW
              The socket is used as a raw socket.
````

# 常见参数

-a (all)显示所有选项，默认不显示LISTEN相关
-t (tcp)仅显示tcp相关选项
-u (udp)仅显示udp相关选项
-n 拒绝显示别名，能显示数字的全部转化成数字。
-l 仅列出有在 Listen (监听) 的服務状态

-p 显示建立相关链接的程序名
-r 显示路由信息，路由表
-e 显示扩展信息，例如uid等
-s 按各个协议进行统计
-c 每隔一个固定时间，执行该netstat命令。

提示：LISTEN和LISTENING的状态只有用-a或者-l才能看到

# 实用命令实例

## 列出所有端口 (包括监听和未监听的)

### 列出所有端口 netstat -a

````shell
# netstat -a | more
 Active Internet connections (servers and established)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State
 tcp        0      0 localhost:30037         *:*                     LISTEN
 udp        0      0 *:bootpc                *:*
 
Active UNIX domain sockets (servers and established)
 Proto RefCnt Flags       Type       State         I-Node   Path
 unix  2      [ ACC ]     STREAM     LISTENING     6135     /tmp/.X11-unix/X0
 unix  2      [ ACC ]     STREAM     LISTENING     5140     /var/run/acpid.socket
````

### 列出所有 tcp 端口 netstat -at

````shell
# netstat -at
 Active Internet connections (servers and established)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State
 tcp        0      0 localhost:30037         *:*                     LISTEN
 tcp        0      0 localhost:ipp           *:*                     LISTEN
 tcp        0      0 *:smtp                  *:*                     LISTEN
 tcp6       0      0 localhost:ipp           [::]:*                  LISTEN
````

###  列出所有 udp 端口 netstat -au

````shell
# netstat -au
 Active Internet connections (servers and established)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State
 udp        0      0 *:bootpc                *:*
 udp        0      0 *:49119                 *:*
 udp        0      0 *:mdns                  *:*
````

## 列出所有处于监听状态的 Sockets

### 只显示监听端口 netstat -l

````shell
# netstat -l
 Active Internet connections (only servers)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State
 tcp        0      0 localhost:ipp           *:*                     LISTEN
 tcp6       0      0 localhost:ipp           [::]:*                  LISTEN
 udp        0      0 *:49119                 *:*
````

### 只列出所有监听 tcp 端口 netstat -lt

````shell
# netstat -lt
 Active Internet connections (only servers)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State
 tcp        0      0 localhost:30037         *:*                     LISTEN
 tcp        0      0 *:smtp                  *:*                     LISTEN
 tcp6       0      0 localhost:ipp           [::]:*                  LISTEN
````

### 只列出所有监听 udp 端口 netstat -lu

````shell
# netstat -lu
 Active Internet connections (only servers)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State
 udp        0      0 *:49119                 *:*
 udp        0      0 *:mdns                  *:*
````

### 只列出所有监听 UNIX 端口 netstat -lx

````shell
# netstat -lx
 Active UNIX domain sockets (only servers)
 Proto RefCnt Flags       Type       State         I-Node   Path
 unix  2      [ ACC ]     STREAM     LISTENING     6294     private/maildrop
 unix  2      [ ACC ]     STREAM     LISTENING     6203     public/cleanup
 unix  2      [ ACC ]     STREAM     LISTENING     6302     private/ifmail
 unix  2      [ ACC ]     STREAM     LISTENING     6306     private/bsmtp
````

## 显示每个协议的统计信息

### 显示所有端口的统计信息 netstat -s

````shell
# netstat -s
 Ip:
 11150 total packets received
 1 with invalid addresses
 0 forwarded
 0 incoming packets discarded
 11149 incoming packets delivered
 11635 requests sent out
 Icmp:
 0 ICMP messages received
 0 input ICMP message failed.
 Tcp:
 582 active connections openings
 2 failed connection attempts
 25 connection resets received
 Udp:
 1183 packets received
 4 packets to unknown port received.
 .....
````

### 显示 TCP 或 UDP 端口的统计信息 netstat -st 或 -su

````shell
# netstat -st 
# netstat -su
````

## 在 netstat 输出中显示 PID 和进程名称 netstat -p

netstat -p 可以与其它开关一起使用，就可以添加 “PID/进程名称” 到 netstat 输出中，这样 debugging 的时候可以很方便的发现特定端口运行的程序。

````
# netstat -pt
 Active Internet connections (w/o servers)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
 tcp        1      0 ramesh-laptop.loc:47212 192.168.185.75:www        CLOSE_WAIT  2109/firefox
 tcp        0      0 ramesh-laptop.loc:52750 lax:www ESTABLISHED 2109/firefox
````

## 在 netstat 输出中不显示主机，端口和用户名 (host, port or user)

当你不想让主机，端口和用户名显示，使用 netstat -n。将会使用数字代替那些名称。

同样可以加速输出，因为不用进行比对查询。

````shell
# netstat -an
````

如果只是不想让这三个名称中的一个被显示，使用以下命令

````shell
# netsat -a --numeric-ports
# netsat -a --numeric-hosts
# netsat -a --numeric-users
````

## 持续输出 netstat 信息

netstat 将每隔一秒输出网络信息。

````shell
# netstat -c
 Active Internet connections (w/o servers)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State
 tcp        0      0 ramesh-laptop.loc:36130 101-101-181-225.ama:www ESTABLISHED
 tcp        1      1 ramesh-laptop.loc:52564 101.11.169.230:www      CLOSING
 tcp        0      0 ramesh-laptop.loc:43758 server-101-101-43-2:www ESTABLISHED
 tcp        1      1 ramesh-laptop.loc:42367 101.101.34.101:www      CLOSING
 ^C
````

## 显示系统不支持的地址族 (Address Families)

````shell
netstat --verbose
````

在输出的末尾，会有如下的信息

````shell
netstat: no support for `AF IPX' on this system.
netstat: no support for `AF AX25' on this system.
netstat: no support for `AF X25' on this system.
netstat: no support for `AF NETROM' on this system.
````

## 显示核心路由信息 netstat -r

````shell
# netstat -r
 Kernel IP routing table
 Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
 192.168.1.0     *               255.255.255.0   U         0 0          0 eth2
 link-local      *               255.255.0.0     U         0 0          0 eth2
 default         192.168.1.1     0.0.0.0         UG        0 0          0 eth2
````

__注意__： 使用 netstat -rn 显示数字格式，不查询主机名称。

## 找出程序运行的端口

并不是所有的进程都能找到，没有权限的会不显示，使用 root 权限查看所有的信息。

````shell
# netstat -ap | grep ssh
 tcp        1      0 dev-db:ssh           101.174.100.22:39213        CLOSE_WAIT  -
 tcp        1      0 dev-db:ssh           101.174.100.22:57643        CLOSE_WAIT  -
````

找出运行在指定端口的进程

````shell
# netstat -an | grep ':80'
````

## 显示网络接口列表

````shell
# netstat -i
 Kernel Interface table
 Iface   MTU Met   RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
 eth0       1500 0         0      0      0 0             0      0      0      0 BMU
 eth2       1500 0     26196      0      0 0         26883      6      0      0 BMRU
 lo        16436 0         4      0      0 0             4      0      0      0 LRU
````

显示详细信息，像是 ifconfig 使用 netstat -ie:

````shell

# netstat -ie
 Kernel Interface table
 eth0      Link encap:Ethernet  HWaddr 00:10:40:11:11:11
 UP BROADCAST MULTICAST  MTU:1500  Metric:1
 RX packets:0 errors:0 dropped:0 overruns:0 frame:0
 TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
 collisions:0 txqueuelen:1000
 RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
 Memory:f6ae0000-f6b00000
````

## IP和TCP分析

### 查看连接某服务最多的的IP地址

````shell
# netstat -nat | grep "192.168.1.15:22" |awk '{print $5}'|awk -F: '{print $1}'|sort|uniq -c|sort -nr|head -20
18 221.136.168.36
3 154.74.45.242
2 78.173.31.236
2 62.183.207.98
2 192.168.1.14
2 182.48.111.215
2 124.193.219.34
2 119.145.41.2
2 114.255.41.30
1 75.102.11.99
````

### TCP各种状态列表

````shell
# netstat -nat |awk '{print $6}'|sort|uniq -c|sort -rn
143 ESTABLISHED
113 TIME_WAIT
36 LISTEN
6 SYN_SENT
1 LAST_ACK
1 Foreign
1 FIN_WAIT1
1 established)
````

基本可以利用`sort|uniq -c|sort -rn|head -20`,来找到一些排序列表，如下面，可以通过nginx的access.log找到访问前10位的ip地址

````shell
awk '{print $1}' access.log |sort|uniq -c|sort -nr|head -10
````

# 参考

https://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316661.html

https://www.cnblogs.com/echo1937/p/6677325.html