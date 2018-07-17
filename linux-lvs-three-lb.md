---
title: LVS 所提供的 IP 负载均衡的三种技术
date: 2014-10-02 10:18:13
tags: [linux,lvs]
categories: linux
---
# LVS和负载均衡简介
LVS是Linux Virtual Server的简写，意即Linux虚拟服务器，是一个虚拟的服务器集群系统。

负载均衡就是有两台或者以上的服务器或者站点为我们提供服务，我们将来自客户端的请求靠某种算法尽量平均分摊到这些集群中，从而避免一台服务器因为负载太高而出现故障。简而言之便是将所有的负载均分到多台服务器中，即使其中某个出现故障，用户也能正常访问，得到服务。

广泛使用的是用软件的方式来实现负载均衡，实现效果不错，并且不需要成本是很多企业选择的方式。

以软件实现的负载均衡有两种方式：
* 基于应用层负载均衡
* 基于IP层负载均衡

文章主要就是介绍基于IP层负载均衡；
<!-- more -->
# 基于 IP 层负载均衡
基于 IP 层负载均衡：用户通过虚拟 IP 地址（Virtual IP Address）访问服务时，访问请求的报文会到达负载调度器，由它进行负载均衡调度，从一组真实服务器选出一个，将报文处理并转发给选定服务器的地址。真实服务器的回应报文经过负载调度器时，将报文的源地址和源端口改为 Virtual IP Address 和相应的端口，再把报文发给用户。

而 IP 的负载技术有以下三种模式：
* 通过NAT实现虚拟服务器（VS/NAT）
* 通过IP隧道实现虚拟服务器（VS/TUN）
* 通过直接路由实现虚拟服务器（VS/DR）

## VS/NAT 实现虚拟服务器
由于 IPv4 中 IP 地址空间的日益紧张和安全方面的原因，很多网络使用保留 IP 地址（10.0.0.0/255.0.0.0、 172.16.0.0/255.128.0.0和192.168.0.0/255.255.0.0）。这些地址不在 Internet 上使用，而是专门为内部网络预留的。

当内部网络中的主机要访问 Internet 或被 Internet 访问时，就需要采用网络地址转换（Network Address Translation, 以下简称NAT），将内部地址转化为 Internet 上可用的外部地址。
NAT 的工作原理是报文头（目标地址、源地址和端口等）被正确改写后，客户相信 它们连接一个 IP 地址，而不同 IP 地址的服务器组也认为它们是与客户直接相连的。由此，可以用 NAT 方法将不同 IP 地址的并行网络服务变成在一个IP地址上的一个虚拟服务。

VS/NAT（Virtual Server via Network Address Translation）实现的虚拟服务器是这样的一个结构，主要经过这样的一些步骤：
[![](http://idiotsky.top/images1/linux-lvs-three-lb-1.jpg)](http://idiotsky.top/images1/linux-lvs-three-lb-1.jpg)
* 客户端通过 Internet 向服务器发起请求，而请求的 IP 地址指向的是调度器上对外公布的 IP 地址；（因为它并不是真正处理请求的服务器 IP 地址，所以称之为 虚拟 IP 地址，简称为 VIP，Virtual IP Address）
* 请求报文到达调度器（Load Balancer），调度器根据调度算法从一组真实的服务器（因为他们是真正处理用户请求的服务器，所以称为真实服务器，Real server。其 IP 地址也被称为真实 IP，简称为 RIP）中选出一台当前负载不高的服务器。然后将客户端的请求报文中的目标地址（Load Balancer 的 VIP）和端口通过 iptables 的 NAT 改写为选定服务器的 IP 地址和服务的端口。最后将修改后的报文发送给选出的服务器。同时，调度器在连接Hash 表中记录这个连接；当这个连接的下一个报文到达时，从连接Hash表中可以得到原选定服务器的地址和端口，进行同样的改写操作，并将报文传给原选定的服务器。
* Real Server 接收到报文之后，做出了响应的处理，然后将响应的报文发送给 Load Balancer；
* Load Balancer 接收到响应的报文时，将报文的源地址和源端口改为Virtual IP Address和相应的端口，再把报文发给用户。

这样，客户所看到的只是在 Virtual IP Address 上提供的服务，而服务器集群的结构对用户是透明的。

下面，举个例子来进一步说明VS/NAT，如图所示：
[![](http://idiotsky.top/images1/linux-lvs-three-lb-2.jpg)](http://idiotsky.top/images1/linux-lvs-three-lb-2.jpg)

VS/NAT 的配置如下表所示，所有到IP地址为205.100.106.2和端口为80的流量都被负载均衡地调度的真实服务器172.16.1.3:80和 172.16.1.4:8080上。目标地址为205.100.106.2:21的报文被转移到172.16.1.3:21上。而到其他端口的报文将被拒绝。

Protocol|	Virtual IP Address|	Port|	Real IP Address	|Port
-----|-----|-----|------|----
TCP|	205.100.106.2|	80|	172.16.1.3	|80
|||172.16.1.4|	8080
TCP|	205.100.106.2	|21	|172.16.1.3	|21

当客户端访问Web服务的时候，报文中可能有以下的源地址和目标地址：

SOURCE	|DEST
----|----
203.100.106.1:3456|	205.100.106.2:80

报文到达调度器之后，调度器从调度列表中选出一台服务器，例如是172.16.1.4:8080。该报文会被改写为如下地址，并将它发送给选出的服务器。

SOURCE	|DEST
----|-----
203.100.106.1:3456	|172.16.1.4:8080

Real Server 收到修改后的报文之后，做出响应，然后将响应报文返回到调度器，报文如下：


SOURCE|	DEST
-----|----
172.16.1.4:8080	|203.100.106.1:3456

响应报文的源地址会被 Load Balacer 改写为虚拟服务的地址，再将报文发送给客户：

SOURCE	|DEST
-----|-----
205.100.106.2:80|	203.100.106.1:3456

这样，客户认为是从202.103.106.5:80服务得到正确的响应，而不会知道该请求是 Real Server1 还是 Real Server2 处理的。

这便是 VS/NAT 的处理数据包的整个过程，它有这样的一些特点：
* 集群节点，也就是 Real Server 与 Load Balacer 必须在同一个 IP 网络中
* Load Balancer 位于 Real Server 与客户端之间，处理进出的所有通信
* RIP 通常是私有地址，仅用于各个集群节点之间的通信。
* Real Server 的网关必须指向 Load Balancer
* 支持端口映射：也就是Real Server 的端口可以自己设定，没有必须是与 Load Balancer 一样

VS/NAT 的优势在于可以做到端口映射，但是 Load Balancer 将可能成为集群的瓶颈。因为所有的出入报文都需要 Load Balancer 处理，请求报文较小不是问题，但是响应报文往往较大，都需要 NAT 转换的话，大流量的时候，Load Balancer 将会处理不过来。一般使用 VS/NAT 的话，处理 Real Server 数量达到 10~20 台左右将是极限，并且效率往往不高。

## VS/DR 实现虚拟服务器
在VS/NAT 的集群系统中，请求和响应的数据报文都需要通过负载调度器，当真实服务器的数目在10台和20台之间时，负载调度器将成为整个集群系统的新瓶颈。大多数 Internet服务都有这样的特点：请求报文较短而响应报文往往包含大量的数据。

既然同时处理进出报文会大大的影响效率，增加机器的负载，那么若是仅仅处理进来的报文，即在负载调度器中只负责调度请求,而出去的报文由 Real Server 直接发给客户端这样岂不是高效许多。

VS/DR（Virtual Server via Direct Routing）利用大多数Internet服务的非对称特点，负载调度器中只负责调度请求，而服务器直接将响应返回给客户，可以极大地提高整个集群 系统的吞吐量。

VS/DR 实现的虚拟服务器是这样的一个结构，主要经过这样的一些步骤：
[![](http://idiotsky.top/images1/linux-lvs-three-lb-3.jpg)](http://idiotsky.top/images1/linux-lvs-three-lb-3.jpg)
* 客户端通过 Internet 向服务器发起请求，而请求的 IP 地址指向的是调度器上对外公布的 IP 地址；
* 请求报文到达调度器（Load Balancer），调度器根据各个服务器的负载情况，动态地选择一台服务器，不修改也不封装IP报文，而是将数据帧的MAC地址改为选出服务器的MAC地址，再将修改后 的数据帧在与服务器组的局域网上发送。因为数据帧的MAC地址是选出的服务器，所以服务器肯定可以收到这个数据帧；
* Real Server 接收到报文之后，发现报文的目标地址 VIP 是在本地的网络设备上，服务器处理这个报文，然后根据路由表将响应报文直接返回给客户。
[![](http://idiotsky.top/images1/linux-lvs-three-lb-4.jpg)](http://idiotsky.top/images1/linux-lvs-three-lb-4.jpg)
在VS/DR中，根据缺省的TCP/IP协议栈处理，请求报文的目标地址为VIP，响应报文的源地址肯定也为VIP，所以响应报文不需要作任何修改，可以直接返回给客户，客户认为得到正常的服务，而不会知道是哪一台服务器处理的。

这便是 VS/DR 的处理数据包的整个过程，它有这样的一些特点：
* 集群节点，也就是 Real Server 与 Load Balacer 必须在同一个物理网络中（若是不同网段的话结构将变得复杂）
* RIP 通常是私有地址，也可以是公网地址，以便于远程管理与监控。
* Load Balancer 仅仅负责处理入站的请求，Real Server 将直接响应客户端
* Real Server 的网关不能指向 Load Balancer
* 不支持端口映射：也就是Real Server 的端口必须是与 Load Balancer 对外服务的一样

## VS/TUN 实现虚拟服务器
VS/DR 限制 Real Server 与 Load Balancer 必须在同一个物理网络中，那若是分散在各地岂不是无法使用？所以有了 VS/TUN（Virtual Server via IP Tunneling）的诞生。

IP隧道（IP tunneling）是将一个IP报文封装在另一个IP报文的技术，这可以使得目标为一个IP地址的数据报文能被封装和转发到另一个IP地址。IP隧道技术亦称为IP封装技术（IP encapsulation）。IP隧道主要用于移动主机和虚拟私有网络（Virtual Private Network），在其中隧道都是静态建立的，隧道一端有一个IP地址，另一端也有唯一的IP地址。

我们利用IP隧道技术将请求报文封装转发给后端服务器，响应报文能从后端服务器直接返回给客户。但在这里，后端服务器有一组而非一个，所以我们不可能静态地建立一一对应的隧道，而是动态地选择 一台服务器，将请求报文封装和转发给选出的服务器。这样，我们可以利用IP隧道的原理将一组服务器上的网络服务组成在一个IP地址上的虚拟网络服务。 VS/TUN的体系结构如图所示，各个服务器将VIP地址配置在自己的IP隧道设备上。
[![](http://idiotsky.top/images1/linux-lvs-three-lb-5.jpg)](http://idiotsky.top/images1/linux-lvs-three-lb-5.jpg)
它的连接调度和管理与VS/NAT中的一样，只是它的报文转发方法不同。调度器根据各个服务器的负载情况，动态地选择一台服务器， 将请求报文封装在另一个 IP 报文中，再将封装后的 IP 报文转发给选出的服务器；服务器收到报文后，先将报文解封获得原来目标地址为 VIP的报文，服务器发现VIP地址被配置在本地的 IP隧道设备上，所以就处理这个请求，然后根据路由表将响应报文直接返回给客户。

这便是 VS/TUN 的处理数据包的整个过程，它有这样的一些特点：
* 集群节点，也就是 Real Server 与 Load Balacer 可以跨越公网
* RIP 必须是公网地址。
* Load Balancer 仅仅负责处理入站的请求，Real Server 将直接响应客户端
* Real Server 的网关不能指向 Load Balancer
* 不支持端口映射：也就是Real Server 的端口必须是与 Load Balancer 对外服务的一样

这便是 LVS 所提供的 IP 负载均衡的三种技术，我们可以根据自己的情况做出不同的选择。