---
title: redis数据结构-整数集合
date: 2017-09-16 12:07:30
tags: [redis,数据结构,整数集合]
categories: redis
---
> 整数集合（intset）是集合键的底层实现之一： 当一个集合只包含整数值元素， 并且这个集合的元素数量不多时， Redis 就会使用整数集合作为集合键的底层实现。
# 整数集合的实现

<!-- more -->