---
title: SELECT * ...... FOR UPDATE 锁机制
date: 2016-12-19 20:35:56
tags: mysql
categories: mysql
---
> 由于InnoDB预设是Row-Level Lock，InnoDB行锁是通过给索引上的索引项加锁来实现的，这一点MySQL与Oracle不同，后者是通过在数据块中对相应数据行加锁来实现的。InnoDB这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB才使用行级锁，否则，InnoDB将使用表锁！ 

<!-- more -->
举个例子: 
假设有个表单products ，里面有id跟name二个栏位，id是主键。 
例1: (明确指定主键，并且有此笔资料，row lock)
````sql
SELECT * FROM products WHERE id='3' FOR UPDATE;
SELECT * FROM products WHERE id='3' and type=1 FOR UPDATE;
````
例2: (明确指定主键，若查无此笔资料，无lock)
````sql
SELECT * FROM products WHERE id='-1' FOR UPDATE;
````
例3: (无主键，table lock)
````sql
SELECT * FROM products WHERE name='Mouse' FOR UPDATE;
````
例4: (主键不明确，table lock)
````sql
SELECT * FROM products WHERE id<>'3' FOR UPDATE;
````
例5: (主键不明确，table lock)
````sql
SELECT * FROM products WHERE id LIKE '3' FOR UPDATE;
````
注1: FOR UPDATE仅适用于InnoDB，且必须在交易区块(BEGIN/COMMIT)中才能生效。 
注2: 要测试锁定的状况，可以利用mysql的Command Mode ，开二个视窗来做测试。

在MySql 5.0中测试确实是这样的 
另外：MyAsim 只支持表级锁，InnerDB支持行级锁 
添加了(行级锁/表级锁)锁的数据不能被其它事务再锁定，也不被其它事务修改（修改、删除） 
是表级锁时，不管是否查询到记录，都会锁定表

此外，如果A与B都对表id进行查询但查询不到记录，则A与B在查询上不会进行row锁，但A与B都会获取排它锁，此时A再插入一条记录的话则会因为B已经有锁而处于等待中，此时B再插入一条同样的数据则会抛出Deadlock found when trying to get lock; try restarting transaction然后释放锁，此时A就获得了锁而插入成功

上面介绍过SELECT ... FOR UPDATE 的用法，不过锁定(Lock)的数据是判别就得要注意一下了。由于InnoDB 预设是Row-Level Lock，所以只有「明确」的指定主键，MySQL 才会执行Row lock (只锁住被选取的数据) ，否则MySQL 将会执行Table Lock (将整个数据表单给锁住)。

转载 http://www.cnblogs.com/chenwenbiao/archive/2012/06/06/2537508.html