---
title: MySQL索引方法
date: 2017-09-20 21:58:53
tags: mysql
categories: mysql
---
MySQL目前主要有以下几种索引方法：B-Tree，Hash，R-Tree。

# B-Tree
B-Tree是最常见的索引类型，所有值（被索引的列）都是排过序的，每个叶节点到跟节点距离相等。所以B-Tree适合用来查找某一范围内的数据，而且可以直接支持数据排序（ORDER BY）
B-Tree在MyISAM里的形式和Innodb稍有不同：
MyISAM表数据文件和索引文件是分离的，索引文件仅保存数据记录的磁盘地址
InnoDB表数据文件本身就是主索引，叶节点data域保存了完整的数据记录
<!-- more -->
# Hash索引
1. 仅支持"=","IN"和"<=>"精确查询，不能使用范围查询：
    由于Hash索引比较的是进行Hash运算之后的Hash值，所以它只能用于等值的过滤，不能用于基于范围的过滤，因为经过相应的Hash算法处理之后的Hash
2. 不支持排序：
    由于Hash索引中存放的是经过Hash计算之后的Hash值，而且Hash值的大小关系并不一定和Hash运算前的键值完全一样，所以数据库无法利用索引的数据来避免任何排序运算
3. 在任何时候都不能避免表扫描：
    由于Hash索引比较的是进行Hash运算之后的Hash值，所以即使取满足某个Hash键值的数据的记录条数，也无法从Hash索引中直接完成查询，还是要通过访问表中的实际数据进行相应的比较，并得到相应的结果
4. 检索效率高，索引的检索可以一次定位，不像B-Tree索引需要从根节点到枝节点，最后才能访问到页节点这样多次的IO访问，所以Hash索引的查询效率要远高于B-Tree索引
5. 只有Memory引擎支持显式的Hash索引，但是它的Hash是nonunique的，冲突太多时也会影响查找性能。Memory引擎默认的索引类型即是Hash索引，虽然它也支持B-Tree索引

# R-Tree索引
R-Tree在MySQL很少使用，仅支持geometry数据类型，支持该类型的存储引擎只有MyISAM、BDb、InnoDb、NDb、Archive几种。
