---
title: 熔断器概念
date: 2018-11-06 22:41:21
tags: 熔断器
categories: 高可用
---

# 前言

分布式系统中经常会出现某个基础服务不可用造成整个系统不可用的情况, 这种现象被称为服务雪崩效应. 为了应对服务雪崩, 一种常见的做法是手动服务降级. 而Hystrix的出现,给我们提供了另一种选择.

# 服务雪崩效应的定义

服务雪崩效应是一种因 服务提供者 的不可用导致 服务调用者 的不可用,并将不可用 逐渐放大 的过程.如果所示:

[![](http://idiotsky.top/images3/CircuitBreaker-1.png)](http://idiotsky.top/images3/CircuitBreaker-1.png)

上图中, A为服务提供者, B为A的服务调用者, C和D是B的服务调用者. 当A的不可用,引起B的不可用,并将不可用逐渐放大C和D时, 服务雪崩就形成了.


<!-- more -->

# 服务雪崩效应形成的原因

我把服务雪崩的参与者简化为 服务提供者 和 服务调用者, 并将服务雪崩产生的过程分为以下三个阶段来分析形成的原因:

1. 服务提供者不可用
2. 重试加大流量
3. 服务调用者不可用

[![](http://idiotsky.top/images3/CircuitBreaker-2.png)](http://idiotsky.top/images3/CircuitBreaker-2.png)

> to be continue...

# 参考

https://segmentfault.com/a/1190000005988895