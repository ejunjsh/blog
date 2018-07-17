---
title: MySQL分区和分表
date: 2016-09-21 23:29:55
tags: mysql
categories: mysql
---
# 概念
1. 为什么要分表和分区？
日常开发中我们经常会遇到大表的情况，所谓的大表是指存储了百万级乃至千万级条记录的表。这样的表过于庞大，导致数据库在查询和插入的时候耗时太长，性能低下，如果涉及联合查询的情况，性能会更加糟糕。分表和表分区的目的就是减少数据库的负担，提高数据库的效率，通常点来讲就是提高表的增删改查效率。
2. 什么是分表？
分表是将一个大表按照一定的规则分解成多张具有独立存储空间的实体表，我们可以称为子表，每个表都对应三个文件，MYD数据文件，.MYI索引文件，.frm表结构文件。这些子表可以分布在同一块磁盘上，也可以在不同的机器上。app读写的时候根据事先定义好的规则得到对应的子表名，然后去操作它。
3. 什么是分区？
分区和分表相似，都是按照规则分解表。不同在于分表将大表分解为若干个独立的实体表，而分区是将数据分段划分在多个位置存放，可以是同一块磁盘也可以在不同的机器。分区后，表面上还是一张表，但数据散列到多个位置了。app读写的时候操作的还是大表名字，db自动去组织分区的数据。
4. mysql分表和分区有什么联系呢？
（1）都能提高mysql的性高，在高并发状态下都有一个良好的表现。
（2）分表和分区不矛盾，可以相互配合的，对于那些大访问量，并且表数据比较多的表，我们可以采取分表和分区结合的方式（如果merge这种分表方式，不能和分区配合的话，可以用其他的分表试），访问量不大，但是表数据很多的表，我们可以采取分区的方式等。
（3）分表技术是比较麻烦的，需要手动去创建子表，app服务端读写时候需要计算子表名。采用merge好一些，但也要创建子表和配置子表间的union关系。
（4）表分区相对于分表，操作方便，不需要创建子表。
<!-- more -->

# 分区
## 分区的类型：
1. Range：把连续区间按范围划分
例：
````sql
create table user(
    id int(11),
    money int(11) unsigned not null,
    date datetime
)
partition by range(YEAR(date))(
    partition p2014 values less than (2015),
    partition p2015 values less than (2016),
    partition p2016 values less than (2017),
    partition p2017 values less than maxvalue
);
````
2. List：把离散值分成集合，按集合划分，适合有固定取值列的表
例：
````sql
create table user(
    a int(11),
    b int(11)
)
partition by list(b)(
    partition p0 values in (1,3,5,7,9),
    partition p1 values in (2,4,6,8,0)
);
````
3. Hash：随机分配，分区数固定
例：
````sql
create table user(
    a int(11),
    b datetime
)
partition by hash(YEAR(b))
partitions 4;
````
4. Key：类似Hash，区别是只支持1列或多列,且mysql提供自身的Hash函数
例：
````sql
create table user(
    a int(11),
    b datetime
)
partition by key(b)
partitions 4;
````

## 分区管理
1. 新增分区
````sql
ALTER TABLE sale_data
ADD PARTITION (PARTITION p201710 VALUES LESS THAN (201711));
````
2. 删除分区
````sql
--当删除了一个分区，也同时删除了该分区中所有的数据。
ALTER TABLE sale_data DROP PARTITION p201710;
````
3. 分区的合并
下面的SQL，将p201701 - p201709 合并为3个分区p2017Q1 - p2017Q3
````sql
ALTER TABLE sale_data
REORGANIZE PARTITION p201701,p201702,p201703,
p201704,p201705,p201706,
p201707,p201708,p201709 INTO
(
    PARTITION p2017Q1 VALUES LESS THAN (201704),
    PARTITION p2017Q2 VALUES LESS THAN (201707),
    PARTITION p2017Q3 VALUES LESS THAN (201710)
);
````

## 分区应该注意的事项：
* 做分区时，要么不定义主键，要么把分区字段加入到主键中。
* 分区字段不能为NULL，要不然怎么确定分区范围呢，所以尽量NOT NULL

# 分表
## 垂直分表
把原来有很多列的表拆分成多个表，原则是：
（1）把常用、不常用的字段分开放
（2）把大字段独立存放在一个表中

## 水平分表
为了解决单表数据量过大的问题，每个水平拆分表的结构完全一致。
### 按时间结构
如果业务系统对时效性较高，比如新闻发布系统的文章表，可以把数据库设计成时间结构，按时间分有几种结构：
（a）平板式
表类似：
````
article_201701
article_201702
article_201703
````
用年来分还是用月可自定，但用日期的话表就太多了，也没这必要。一般建议是按月分就可以。
这种分法，其难处在于，假设我要列20条数据，结果这三张表里都有2条，那么业务上很有可能要求读三次表。如果时间长了，有几十张表，而每张表是0条，那不就是要读完整个系统的表才行么?另外这个结构，要作分页是比较难实现的。
主键：在这个系统中，主键是13位带毫秒的时间戳，不要用自动编号，否则难以通过主键定位到表，也可以在查询时带上时间，但比较烦琐。
（b）归档式
表类似：
````
article_old
article_new
````
为了解决平板式的缺点，可以采用时间归档式设计，可以看到这个系统只有两张表。一张是旧文章表，一张是新文章表，新文章表放2个月的信息，每天定期把2个月中的最早一天的文章归入旧表中。这样一方面可以解决性能问题，因为一般新闻发布系统读取的都是新的内容，旧的内容读取少;第二可以委婉地解决功能问题，比如平板式所说的问题，在归档式中最多也只需要读2张表就完成了。
归档式的缺点在于旧表容量还是相对比较大，如果业务允许，可对旧表中的超旧内容进行再归档或直接清理掉。
### 按版块结构
如果按照文章的所属版块进行拆表，比如新闻、体育版块拆表，一方面可以使每个表数据量分离，另一方面是各版块之间相互影响可降到最低。假如新闻版块的数据表损坏或需要维护，并不会影响到体育版块的正常工作，从而降低了风险。版块结构同时常用于bbs这样的系统。
板块结构也有几种分法：
（a）对应式
对于版块数量不多，而且较为固定的形式，就直接对应就好。比如新闻版块，可以分出新闻的目录表，新闻的文章表等。
````
news_category
news_article
sports_category
sports_article
````
可看到每一个版块都对应着一组相同的表结构，好处就是一目了然。在功能上，因为版块之间还是有一些隔阂，所以需要联合查询的需求不多，开发上比时间结构的方式要轻松。
主键：依旧要考虑的，在这个系统中，主键是版块+时间戳，单纯的时间戳或自动编号也能用，查询时要记得带上版块用于定位表。
（b）冷热式
对应式的缺点是，如果版块数量很大而且不确定，那要分出的表数量就太多了。举个例子：百度贴吧，如果按一个词条一个表设计，那得有多少张表呢?
用这样的方式吧。
````
tieba_汽车
tieba_飞机
tieba_火箭
tieba_unite
````
这个表汽车、火箭表是属于热门表，定义为新建的版块放在unite表里面，待到其超过一万张主贴的时候才开对应表结构。因为在贴吧这种系统中，冷门版块
肯定比热门版块多得多，这些冷门版块通常只有几张帖子，为它们开表也太浪费了;同时热门版块数量和访问量等，又比冷门版块多得多，非常有特点。
unite表还可以扩展成哈希表，利用词条的md5编码，可以分成n张表，我算了一下，md5前一位可分36张表，两位即是1296张表，足够了。
````
tieba_unite_ab
tieba_unite_ac
````
### 按哈希结构
哈希结构通常用于博客之类的基于用户的场合，在博客这样的系统里有几个特点，1是用户数量非常多，2是每个用户发的文章数量都较少，3是用户发文章不定
期，4是每个用户发得不多，但总量仍非常之大。基于这些特点，用以上所说的任何一种分表方式都不合适，一没有固定的时效不宜用时间拆，二用户很多，而且还
偏偏都是冷门，所以也不宜用版块(用户)拆。
哈希结构在上面有所提及，既然按每个用户不好直接拆，那就把一群用户归进一个表好了。
````
blog_aa
blog_ab
blog_ac
````
如上所说，md5取前两位哈希可以达到1296张表，如果觉得不够，那就再加一位，总数可达46656张表，还不够?
表的数量太多，要创建这些表也是挺麻烦的，可以考虑在程序里往数据库insert之前，多执行一句判断表存在与否并创建表的语句，很实用，消耗也并不很大。
主键：依旧要考虑的，在这个系统中，主键是用户ID+时间戳，单纯的时间戳或自动编号也能用，但查询时要记得带上用户名用于定位表。

## merge存储引擎分表
### 使用场景
Merge表有点类似于视图。使用Merge存储引擎实现MySQL分表，这种方法比较适合那些没有事先考虑分表，随着数据的增多，已经出现了数据查询慢的情况。

这个时候如果要把已有的大数据量表分开比较痛苦，最痛苦的事就是改代码。所以使用Merge存储引擎实现MySQL分表可以避免改代码。

　　Merge引擎下每一张表只有一个MRG文件。MRG里面存放着分表的关系，以及插入数据的方式。它就像是一个外壳，或者是连接池，数据存放在分表里面。

对于增删改查，直接操作总表即可。

### 建表
1. 用户1表
````sql
CREATE TABLE `user1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) DEFAULT NULL,
  `sex` int(1) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8; 
````
2. 用户2表
````sql
create table user2 like user1;
````
3. 主表
````sql
CREATE TABLE `alluser` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) DEFAULT NULL,
  `sex` int(1) NOT NULL DEFAULT '0',
  KEY `id` (`id`)
) ENGINE=MRG_MyISAM DEFAULT CHARSET=utf8 INSERT_METHOD=LAST UNION=(`user1`,`user2`);
````
1) ENGINE = MERGE 和 ENGINE = MRG_MyISAM是一样的意思，都是代表使用的存储引擎是 Merge。
2) INSERT_METHOD，表示插入方式，取值可以是：0 和 1，0代表不允许插入，1代表可以插入；
3) FIRST插入到UNION中的第一个表，LAST插入到UNION中的最后一个表。

### 操作
1. 先在user1表中增加一条数据，然后再在user2表中增加一条数据，查看 alluser中的数据。
````sql
insert into user1(name,sex) values ('张三',1);
insert into user2(name,sex) values ('李四',2);
select * from alluser;  
````
发现是刚刚插入的数据如下：
[![](http://idiotsky.top/images/mysql-partition-1.png)](http://idiotsky.top/images/mysql-partition-1.png)
这就出现了一个id重复，这就造成了当删除和修改的时候异常，解决办法是给 alluser的id赋唯一值。
我们解决方法是，重新建立一张表tb_ids(id int)，用来专门存一个id的，并插入一条初始数据，同时删除掉user1和user2中的数据。
````sql
create table tb_ids(id int);
insert into tb_ids values(1);
delete from user1;
delete from user2;
````
然后在user1和user2表中建立一个触发器，触发器的功能是 当在user1或者user2表中增加一条记录时，取出tb\_ids中的id值，赋给user1和user2的id，然后将tb_ids的id值加1，
触发器内容如下（将user1改为user2）：　　　
````sql
DELIMITER $$
   CREATE TRIGGER tr_seq
   BEFORE INSERT on user1
   FOR EACH ROW BEGIN 
      select id  into @testid from tb_ids limit 1;
      update tb_ids set id = @testid + 1;
   set new.id =  @testid;
   END$$
   DELIMITER;
````
2. 在user1和user2表中分别增加一条数据，
````sql
insert into user1(name,sex) values('王五',1);
insert into user2(name,sex) values('赵六',2);
````
3. 查询user1和user2中的数据：
[![](http://idiotsky.top/images/mysql-partition-2.png)](http://idiotsky.top/images/mysql-partition-2.png)
[![](http://idiotsky.top/images/mysql-partition-3.png)](http://idiotsky.top/images/mysql-partition-3.png)
4. 查询总表alluser中的数据，发现id没有重复的：
[![](http://idiotsky.top/images/mysql-partition-4.png)](http://idiotsky.top/images/mysql-partition-4.png)