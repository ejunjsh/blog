---
title: MySQL优化原理
date: 2017-09-29 23:20:35
tags: mysql
categories: mysql
---
说起MySQL的查询优化，相信大家收藏了一堆奇技淫巧：不能使用SELECT *、不使用NULL字段、合理创建索引、为字段选择合适的数据类型..... 你是否真的理解这些优化技巧？是否理解其背后的工作原理？在实际场景下性能真有提升吗？我想未必。因而理解这些优化建议背后的原理就尤为重要，希望本文能让你重新审视这些优化建议，并在实际业务场景下合理的运用。

# MySQL逻辑架构
如果能在头脑中构建一幅MySQL各组件之间如何协同工作的架构图，有助于深入理解MySQL服务器。下图展示了MySQL的逻辑架构图。
[![](http://idiotsky.me/images1/mysql-optimization-mechanism-1.jpg)](http://idiotsky.me/images1/mysql-optimization-mechanism-1.jpg)
<!-- more -->
MySQL逻辑架构整体分为三层，最上层为客户端层，并非MySQL所独有，诸如：连接处理、授权认证、安全等功能均在这一层处理。

MySQL大多数核心服务均在中间这一层，包括查询解析、分析、优化、缓存、内置函数(比如：时间、数学、加密等函数)。所有的跨存储引擎的功能也在这一层实现：存储过程、触发器、视图等。

最下层为存储引擎，其负责MySQL中的数据存储和提取。和Linux下的文件系统类似，每种存储引擎都有其优势和劣势。中间的服务层通过API与存储引擎通信，这些API接口屏蔽了不同存储引擎间的差异。

# MySQL查询过程
我们总是希望MySQL能够获得更高的查询性能，最好的办法是弄清楚MySQL是如何优化和执行查询的。一旦理解了这一点，就会发现：很多的查询优化工作实际上就是遵循一些原则让MySQL的优化器能够按照预想的合理方式运行而已。

当向MySQL发送一个请求的时候，MySQL到底做了些什么呢？
[![](http://idiotsky.me/images1/mysql-optimization-mechanism-2.jpg)](http://idiotsky.me/images1/mysql-optimization-mechanism-2.jpg)

> to be continue...