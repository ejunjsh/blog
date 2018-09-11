---
title: MySql中的锁机制
date: 2018-09-10 17:51:52
tags: [悲观锁,乐观锁,共享锁,排他锁,间隙锁,意向锁,mysql]
categories: mysql
---

# 乐观锁与悲观锁

* 乐观锁：每次读数据的时候都认为其他人不会修改，所以不会上锁，而是在更新的时候去判断在此期间有没有其他人更新了数据，可以使用版本号机制。在数据库中可以通过为数据表增加一个版本号字段实现。读取数据时将版本号一同读出，数据每次更新时对版本号加一。当我们更新的时候，判断数据库表对应记录的当前版本号与第一次取出来的版本号值进行比对，如果值相等，则予以更新，否则认为是过期数据。乐观锁适用于多读的应用类型，可以提高吞吐量。
* 悲观锁：每次读数据的时候都认为别人会修改，所以每次在读数据的时候都会上锁，这样别人想读这个数据时就会被阻塞。MySQL中就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在操作之前先上锁。

<!-- more -->
# 共享锁与排他锁

* 共享锁：共享锁又叫做读锁或S锁，加上共享锁后在事务结束之前其他事务只能再加共享锁、只能对其进行读操作不能写操作，除此之外其他任何类型的锁都不能再加了。
````sql
# 加上lock in share mode
SELECT description FROM book_book lock in share mode;
````
* 排他锁：排他锁又叫写锁或X锁，某个事务对数据加上排他锁后，只能这个事务对其进行读写，在此事务结束之前，其他事务不能对其加任何锁，可以读取,不能进行写操作，需等待其释放。
````sql
# 加上for update
SELECT description FROM book_book for update; 
````

# 行锁与表锁

> 行锁与表锁区别在于锁的粒度，在Innodb引擎中既支持行锁也支持表锁(MyISAM引擎只支持表锁)，只有通过索引条件检索数据InnoDB才使用行级锁，否则，InnoDB将使用表锁。

* 表锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突概率高，并发度最低
* 行锁：开销大，加锁慢；会出现死锁；锁定粒度小，发生锁冲突的概率低，并发度高

下面举两个例子说明上面几种锁：

````sql
# 事务1
BEGIN;
SELECT description FROM book_book where name = 'JAVA编程思想' lock in share mode;

# 事务2
BEGIN;
UPDATE book_book SET name = 'new book' WHERE name = 'new';

# 查看事务状态
SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;

trx_id  trx_state       trx_started           trx_tables_locked    trx_rows_locked
39452	LOCK WAIT	2018-09-08 19:01:39	    1	                1	
282907511143936	RUNNING	2018-09-08 18:58:47	    1	                38	
````

事务1给book表加上了共享锁，事务2尝试修改book表发生了阻塞，查看事务状态可以知道事务一由于没有走索引使用了表锁。

````sql
# 事务1
BEGIN;
SELECT description FROM book_book WHERE id = 2 lock in share mode;

# 事务2
BEGIN;
UPDATE book_book SET name = 'new book' WHERE id = 1; 

# 查看事务状态
SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;

trx_id          trx_state   trx_started     trx_tables_locked    trx_rows_locked
39454	        RUNNING	2018-09-08 19:10:44	1	                1	
282907511143936	RUNNING	2018-09-08 19:10:35	1	                1	
````

事务1给book表加上了共享锁，事务2尝试修改book表并没有发生阻塞。这是由于事务一和事务二都走了索引，所以使用的是行锁，并不会发生阻塞。

# 意向锁(InnoDB特有)

> 意向锁的意义在于方便检测表锁和行锁之间的冲突

* 意向锁：意向锁是一种表级锁，代表要对某行记录进行操作。分为意向共享锁(IS)和意向排他锁(IX)。
* 行锁和表锁之间的冲突：事务A给表中的某一行加了共享锁，让这一行只能读不能写。之后事务B申请整个表的排他锁。如果事务B申请成功，那么它就能修改表中的任意一行，这与A持有的行锁是冲突的。InnoDB引入了意向锁来判断它们之间的冲突。
    * 没有意向锁的情况：1、判断表是否已被其他事务用表锁锁表。2、判断表中的每一行是否已被行锁锁住，这样要遍历整个表，效率很低。
    * 意向锁存在的情况：1、判断表是否已被其他事务用表锁锁表。2、判断表上是否有意向锁
* 意向锁存在时申请锁：申请意向锁的动作是数据库完成的，上述例子中事务A申请一行的行锁的时候，数据库会自动先开始申请表的意向锁，当事务B申请表的排他锁时检测到存在意向锁则会阻塞。
* 意向锁会不会存在冲突： 意向锁之间不会冲突, 因为意向锁只是代表要对某行记录进行操作。

# 各种锁之间的共存情况

````
       IX     IS       X      S
IX    兼容    兼容    冲突    冲突
IS    兼容    兼容    冲突    兼容
X     冲突    冲突    冲突    冲突
S     冲突    兼容    冲突    兼容
````

# 死锁

* 概念：两个或两个以上的事务在执行过程中，因争夺资源而造成的一种互相等待的现象。
* 存在条件：1、 互斥条件：一个资源每次只能被一个事务使用。2、 请求与保持条件：一个事务因请求资源而阻塞时，对已获得的资源保持不放。3、不剥夺条件：已获得的资源，在末使用完之前不能强行剥夺。4、循环等待条件：形成一种头尾相接的循环等待关系
* 解除正在死锁的状态：撤销其中一个事务

# 间隙锁(Next-Key锁)

> 间隙锁使得InnoDB解决幻读问题，加上MVCC使得InnoDB的RR隔离级别实现了串行化级别的效果，并且保留了比较好的并发性能。

定义：当我们用范围条件检索数据时请求共享或排他锁时，InnoDB会给符合条件的已有数据的索引加锁；对于键值在条件范围内但并不存在的记录，叫做间隙(GAP)，InnoDB也会对这个"间隙"加锁，这种锁机制就是间隙锁。

例如：book表中存在bookId 1-80，90-99的记录。SELECT * FROM book WHERE bookId < 100 FOR UPDATE。InnoDB不仅会对bookId值为1-80，90-99的记录加锁，也会对bookId在81-89之间(这些记录并不存在)的间隙加锁。这样就能避免事务隔离级别可重复读下的幻读。

# MVCC(多版本并发控制)

> MVCC (Multiversion Concurrency Control)，即多版本并发控制技术,它使得大部分支持行锁的事务引擎，不再单纯的使用行锁来进行数据库的并发控制，取而代之的是把数据库的行锁与行的多个版本结合起来，只需要很小的开销,就可以实现非锁定读，从而大大提高数据库系统的并发性能

## MVCC实现原理

innodb MVCC主要是为Repeatable-Read事务隔离级别做的。在此隔离级别下，A、B客户端所示的数据相互隔离，互相更新不可见

了解innodb的行结构、Read-View的结构对于理解innodb mvcc的实现由重要意义

innodb存储的最基本row中包含一些额外的存储信息 DATA_TRX_ID，DATA_ROLL_PTR，DB_ROW_ID，DELETE BIT

* 6字节的DATA_TRX_ID 标记了最新更新这条行记录的transaction id，每处理一个事务，其值自动+1
* 7字节的DATA_ROLL_PTR 指向当前记录项的rollback segment的undo log记录，找之前版本的数据就是通过这个指
* 6字节的DB_ROW_ID，当由innodb自动产生聚集索引时，聚集索引包括这个DB_ROW_ID的值，否则聚集索引中不包括这个值.，这个用于索引当中
* DELETE BIT位用于标识该记录是否被删除，这里的不是真正的删除数据，而是标志出来的删除。真正意义的删除是在commit的时候


[![](http://idiotsky.top/images3/mysql-lock.png)](http://idiotsky.top/images3/mysql-lock.png)

具体的执行过程

begin->用排他锁锁定该行->记录redo log->记录undo log->修改当前行的值，写事务编号，回滚指针指向undo log中的修改前的行

上述过程确切地说是描述了UPDATE的事务过程，其实undo log分insert和update undo log，因为insert时，原始的数据并不存在，所以回滚时把insert undo log丢弃即可，而update undo log则必须遵守上述过程

下面分别以 **select**、**delete**、 **insert**、 **update**语句来说明

**SELECT**

Innodb检查每行数据，确保他们符合两个标准：

1.InnoDB只查找版本早于当前事务版本的数据行(也就是数据行的版本必须小于等于事务的版本)，这确保当前事务读取的行都是事务之前已经存在的，或者是由当前事务创建或修改的行
2.行的删除操作的版本一定是未定义的或者大于当前事务的版本号，确定了当前事务开始之前，行没有被删除

符合了以上两点则返回查询结果。

**INSERT**

InnoDB为每个新增行记录当前系统版本号作为创建ID。

**DELETE**

InnoDB为每个删除行的记录当前系统版本号作为行的删除ID。

**UPDATE**

InnoDB复制了一行。这个新行的版本号使用了系统版本号。它也把系统版本号作为了删除行的版本。

**说明**

insert操作时 “创建时间”=DB_ROW_ID，这时，“删除时间 ”是未定义的；

update时，复制新增行的“创建时间”=DB_ROW_ID，删除时间未定义，旧数据行“创建时间”不变，删除时间=该事务的DB_ROW_ID；

delete操作，相应数据行的“创建时间”不变，删除时间=该事务的DB_ROW_ID；

select操作对两者都不修改，只读相应的数据

## 对于MVCC的总结

上述更新前建立undo log，根据各种策略读取时非阻塞就是MVCC，undo log中的行就是MVCC中的多版本，这个可能与我们所理解的MVCC有较大的出入，一般我们认为MVCC有下面几个特点：
* 每行数据都存在一个版本，每次数据更新时都更新该版本
* 修改时Copy出当前版本随意修改，各个事务之间无干扰
* 保存时比较版本号，如果成功（commit），则覆盖原记录；失败则放弃copy（rollback）

就是每行都有版本号，保存时根据版本号决定是否成功，听起来含有乐观锁的味道，而Innodb的实现方式是：
* 事务以排他锁的形式修改原始数据
* 把修改前的数据存放于undo log，通过回滚指针与主数据关联
* 修改成功（commit）啥都不做，失败则恢复undo log中的数据（rollback）

二者最本质的区别是，当修改数据时是否要排他锁定，如果锁定了还算不算是MVCC？ 
 
Innodb的实现真算不上MVCC，因为并没有实现核心的多版本共存，undo log中的内容只是串行化的结果，记录了多个事务的过程，不属于多版本共存。但理想的MVCC是难以实现的，当事务仅修改一行记录使用理想的MVCC模式是没有问题的，可以通过比较版本号进行回滚；但当事务影响到多行数据时，理想的MVCC据无能为力了。
 
比如，如果Transaciton1执行理想的MVCC，修改Row1成功，而修改Row2失败，此时需要回滚Row1，但因为Row1没有被锁定，其数据可能又被Transaction2所修改，如果此时回滚Row1的内容，则会破坏Transaction2的修改结果，导致Transaction2违反ACID。
 
理想MVCC难以实现的根本原因在于企图通过乐观锁代替二段提交。修改两行数据，但为了保证其一致性，与修改两个分布式系统中的数据并无区别，而二提交是目前这种场景保证一致性的唯一手段。二段提交的本质是锁定，乐观锁的本质是消除锁定，二者矛盾，故理想的MVCC难以真正在实际中被应用，Innodb只是借了MVCC这个名字，提供了读的非阻塞而已。

## 快照读和当前读

在一个支持MVCC并发控制的系统中，哪些读操作是快照读？哪些操作又是当前读呢？以MySQL InnoDB为例：

快照读：简单的select操作，属于快照读，不加锁。(当然，也有例外，下面会分析)

````
select * from table where ?;
````

当前读：特殊的读操作，插入/更新/删除操作，属于当前读，需要加锁。
````
select * from table where ? lock in share mode;
select * from table where ? for update;
insert into table values (…);
update table set ? where ?;
delete from table where ?;
````
所有以上的语句，都属于当前读，读取记录的最新版本。并且，读取之后，还需要保证其他并发事务不能修改当前记录，对读取记录加锁。其中，除了第一条语句，对读取记录加S锁 (共享锁)外，其他的操作，都加的是X锁 (排它锁)。

# 参考
https://juejin.im/post/5b9332685188255c432279aa
https://www.cnblogs.com/chenpingzhao/p/5041968.html
https://www.cnblogs.com/chenpingzhao/p/5065316.html