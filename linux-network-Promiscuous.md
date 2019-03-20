# linux网卡混杂模式

混杂模式就是接收所有经过网卡的数据包，包括不是发给本机的包，即不验证MAC地址。普通模式下网卡只接收发给本机的包（包括广播包）传递给上层程序，其它的包一律丢弃。
一般来说，混杂模式不会影响网卡的正常工作，多在网络监听工具上使用。

# 网卡具有如下的几种工作模式：
1) 广播模式（Broad Cast Model）:它的物理地址（MAC）地址是 0Xffffff 的帧为广播帧，工作在广播模式的网卡接收广播帧。
2）多播传送（MultiCast Model）：多播传送地址作为目的物理地址的帧可以被组内的其它主机同时接收，而组外主机却接收不到。但是，如果将网卡设置为多播传送模式，它可以接收所有的多播传送帧，而不论它是不是组内成员。
3）直接模式（Direct Model）:工作在直接模式下的网卡只接收目地址是自己 Mac地址的帧。
4）混杂模式（Promiscuous Model）:工作在混杂模式下的网卡接收所有的流过网卡的帧，信包捕获程序就是在这种模式下运行的。

网卡的缺省工作模式包含广播模式和直接模式，即它只接收广播帧和发给自己的帧。如果采用混杂模式，一个站点的网卡将接受同一网络内所有站点所发送的数据包这样就可以到达对于网络信息监视捕获的目的。

 

1，未设置支持promisc

[root@bogon libpcap-1.3.0]# ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.18  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::20c:29ff:fe90:90e9  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:90:90:e9  txqueuelen 1000  (Ethernet)
        RX packets 1529593  bytes 116632252 (111.2 MiB)
        RX errors 0  dropped 13  overruns 0  frame 0
        TX packets 260  bytes 57720 (56.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

 

2，设置支持promisc

[root@bogon libpcap-1.3.0]# ifconfig eth0 promisc 

 

3，已设置支持promisc

[root@bogon libpcap-1.3.0]# ifconfig eth0
eth0: flags=4419<UP,BROADCAST,RUNNING,PROMISC,MULTICAST>  mtu 1500
        inet 192.168.1.18  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::20c:29ff:fe90:90e9  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:90:90:e9  txqueuelen 1000  (Ethernet)
        RX packets 1534849  bytes 117018556 (111.5 MiB)
        RX errors 0  dropped 14  overruns 0  frame 0
        TX packets 262  bytes 58237 (56.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

 

4，设置不支持promisc

[root@bogon libpcap-1.3.0]# ifconfig eth0 -promisc
