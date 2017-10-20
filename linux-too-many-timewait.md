---
title: linux TIME_WAIT过多问题
date: 2017-10-18 20:08:15
tags: [linux,tcp]
categories: linux
---
# 问题
````shell
$ netstat -an |awk '/tcp/ {++S[$NF]}END {for (a in S) print a , S[a]}'
TIME_WAIT 41
CLOSE_WAIT 1
ESTABLISHED 2
LISTEN 7
````
可以看到，TIME_WAIT很多，此状态下的socket不能被回收使用，严重影响服务器的处理能力,甚至耗尽可用的socket,停止服务。
<!-- more -->
# 原因
通信双方建立TCP连接后，主动关闭连接的一方就会进入TIME_WAIT状态
客户端主动关闭连接时，会发送最后一个ack后，然后会进入TIME_WAIT状态，再停留2个MSL时间(后有MSL的解释)，进入CLOSED状态。
下图是以客户端主动关闭连接为例，说明这一过程的:
为什么TIME_WAIT状态还需要等2MSL后才能返回到CLOSED状态？
这是因为：虽然双方都同意关闭连接了，而且握手的4个报文也都协调和发送完毕，按理可以直接回到CLOSED状态（就好比从SYN\_SEND状态到 ESTABLISH状态那样）；但是因为我们必须要假想网络是不可靠的，你无法保证你最后发送的ACK报文会一定被对方收到，因此对方处于 LAST\_ACK状态下的SOCKET可能会因为超时未收到ACK报文，而重发FIN报文，所以这个TIME\_WAIT状态的作用就是用来重发可能丢失的 ACK报文，并保证于此。
[![](http://idiotsky.me/images1/linux-too-many-timewait-1.jpg)](http://idiotsky.me/images1/linux-too-many-timewait-1.jpg)

# linux网络连接状态
[![](http://idiotsky.me/images1/linux-too-many-timewait-2.png)](http://idiotsky.me/images1/linux-too-many-timewait-2.png)
* LISTEN:socket正在监听连接请求。
* ESTABLISHED:socket已经建立连接，表示处于连接的状态
* SYN_SENT:socket正在积极尝试建立一个连接，即处于发送后连接前的一个等待但未匹配进入连接的状态。（主动连接）
* SYN_RECV:已经从网络上收到一个连接请求。（被动连接）
* FIN_WAIT1:socket已关闭，连接正在或正要关闭。(主动关闭)
* FIN_WAIT2:连接已关闭，并且socket正在等待远端结束。
* TIME_WAIT:socket正在等待关闭处理仍在网络上的数据包
* CLOSE_WAIT:远端已经结束，等待socket关闭。(被动关闭)
* LAST_ACK：远端已经结束，并且socket也已关闭，等待acknowledgement。

# linux优化
Linux有很多关于tcp的系统参数，可以有效的优化tcp。
````shell
net.ipv4.tcp_syncookies = 1 #表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击(sync flood 攻击)，默认为0，表示关闭；
net.ipv4.tcp_tw_reuse = 1 #表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1 #表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
net.ipv4.tcp_fin_timeout = 30 #表示如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间。
net.ipv4.tcp_keepalive_time = 1200 #表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为20分钟。
net.ipv4.ip_local_port_range = 1024 65000 #表示用于向外连接的端口范围。缺省情况下很小：32768到61000，改为1024到65000。
net.ipv4.tcp_max_syn_backlog = 8192 #表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。
net.ipv4.tcp_max_tw_buckets = 5000 #表示系统同时保持TIME_WAIT套接字的最大数量，如果超过这个数字，TIME_WAIT套接字将立刻被清除并打印警告信息。默 认为180000，改为5000。对于Apache、Nginx等服务器，上几行的参数可以很好地减少TIME_WAIT套接字数量，但是对于Squid，效果却不大。此项参数可以控制TIME_WAIT套接字的最大数量，避免Squid服务器被大量的TIME_WAIT套接字拖死。
````

# 问题
上面的优化，改了些参数,特别是`tcp_tw_reuse`和`tcp_tw_recycle`这两个参数，确实能够明显看到time_wait减少，甚至基本看不到了。其实这个看上去解决了问题，但是其实还会引发其他的问题。
TCP有一种行为，可以缓存每个主机最新的时间戳，后续请求中如果时间戳小于缓存的时间戳，即视为无效，相应的数据包会被丢弃。
Linux是否启用这种行为取决于`tcp_timestamps`和`tcp_tw_recycle`，因为`tcp_timestamps`缺省就是开启的，所以当`tcp_tw_recycle`被开启后，实际上这种行为就被激活了，当客户端或服务端以NAT方式构建的时候就可能出现问题，下面以客户端NAT为例来说明：
当多个客户端通过NAT方式联网并与服务端交互时，服务端看到的是同一个IP，也就是说对服务端而言这些客户端实际上等同于一个，可惜由于这些客户端的时间戳可能存在差异，于是乎从服务端的视角看，便可能出现时间戳错乱的现象，进而直接导致时间戳小的数据包被丢弃。如果发生了此类问题，具体的表现通常是是客户端明明发送的SYN，但服务端就是不响应ACK，我们可以通过下面命令来确认数据包不断被丢弃的现象：
````shell
$ netstat -s | grep timestamp
... packets rejects in established connections because of timestamp
````
安全起见，通常要禁止`tcp_tw_recycle`。到这里，大家可能会想到另一种解决方案：把`tcp_timestamps`设置为0，`tcp_tw_recycle`设置为1，这样不就可以鱼与熊掌兼得了么？可惜一旦关闭了`tcp_timestamps`，那么即便打开了`tcp_tw_recycle`，也没有效果。

好在我们还有另一个内核参数`tcp_max_tw_buckets`（一般缺省是180000）可用：
````shell
$ sysctl net.ipv4.tcp_max_tw_buckets=100000
````
通过设置它，系统会将多余的TIME_WAIT删除掉，此时系统日志里可能会显示：「TCP: time wait bucket table overflow」，不过除非不得已，否则不要轻易使用。

# 总结
通过上面的问题，解决time_wait过多，好像没有什么完美的方案，所以，还是看情况使用Linux参数。


参考
https://huoding.com/2012/01/19/142
http://blog.csdn.net/dog250/article/details/13760985
http://www.cnblogs.com/bass6/p/5794859.html