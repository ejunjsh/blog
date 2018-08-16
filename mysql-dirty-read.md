---
title: mysql 幻读实验
date: 2018-08-08 20:24:11
tags: [mysql]
categories: mysql
---

# 启动一个mysql

用docker很容易就起一个mysql的环境了，我的[github repo docker-code](https://github.com/ejunjsh/docker-code),有例子

````bash
cd mysql
sudo docker-compose up
````
<!-- more -->

# 登陆mysql

用下面命令可以确认mysql所在的容器，在我机子是`mysql_db_1`
````bash
sudo docker ps
[sudo] password for sky:
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
c7f451f632c0        adminer             "entrypoint.sh doc..."   2 weeks ago         Up 2 minutes        0.0.0.0:8080->8080/tcp   mysql_adminer_1
e2645139c23b        mysql               "docker-entrypoint..."   2 weeks ago         Up 2 minutes        3306/tcp                 mysql_db_1
````

进入容器

````bash
sudo docker exec -it mysql_db_1 bash
mysql -p
````
密码默认是`example`

# 创建测试数据库和表

````sql
mysql> create database test;
mysql> use test;
mysql> DROP TABLE IF EXISTS `tx`;
mysql> CREATE TABLE `tx` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `num` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
````

# 开始实验

mysql的默认隔离级别为`可重复读`，所以是会出现`幻读`的情况的。

还是验证下

````sql
mysql> SELECT @@transaction_isolation;
+-------------------------+
| @@transaction_isolation |
+-------------------------+
| REPEATABLE-READ         |
+-------------------------+
1 row in set (0.01 sec)
````

以前是`SELECT @@tx_isolation;`,新版本要用`SELECT @@transaction_isolation;`

接下来插入一条数据
````sql
mysql> insert into tx (num) values(100);
````

然后开启事务
````sql
mysql> start transaction;
````

更新这条记录

````sql
mysql> update tx set num=200 where num=100;
Rows matched: 1  Changed: 1  Warnings: 0
````

留意上面的`matched`是1的。

先不提交这个事务，开另外个终端按照上面的方法再打开个mysql,之后也开一个事务

````bash
mysql> use test;
mysql> start transaction;
mysql> select * from tx;
+----+-----+
| id | num |
+----+-----+
|  1 | 100 |
+----+-----+
1 row in set (0.00 sec)
````

由于隔离级别，所以看不到对方更新的

然后回到第一个终端,提交事务

````sql
mysql> commit;
````

再回到第二终端，

````sql
mysql> select * from tx;
+----+-----+
| id | num |
+----+-----+
|  1 | 100 |
+----+-----+
1 row in set (0.00 sec)
````

还是因为隔离级别，还是看不到对方更新的。

接下来，重头戏来了，尝试在第二个终端上执行

````sql
mysql> update tx set num=300 where num=100;
Query OK, 0 rows affected (0.00 sec)
Rows matched: 0  Changed: 0  Warnings: 0

mysql> select * from tx;
+----+-----+
| id | num |
+----+-----+
|  1 | 100 |
+----+-----+
1 row in set (0.00 sec)
````

居然更新不了，然后就`幻读`了。这个很容易理解，在第二个终端的事务里，看到都是它启动事务那一刻的快照`snapshot`,所以看不到其他事务的东西，可是一旦更新的时候，就会因为别人事务改变了原来的值，自己没办法再更新它以为的那个值了，所以这种以为就称为了`幻觉(phantom)`，俗称`幻读`。

那假如上面我不指定条件呢 

````sql
mysql> update tx set num=300;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
````

是可以更新的，因为这是全表更新，所以没问题。

````sql
mysql> commit;
````
提交下，方便下面继续做实验

虽说这个`幻读`是问题，但是它也是人们用来做数据库`CAS`的保证吧。

# 数据库`CAS`

我们在上面的表基础上再加一列`version`

````sql
mysql> alter table tx add column version int null;
Query OK, 0 rows affected (0.16 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> select * from tx;
+----+-----+---------+
| id | num | version |
+----+-----+---------+
|  1 | 300 |    NULL |
+----+-----+---------+
1 row in set (0.00 sec)
````

设置个初始值

````sql
mysql> update tx set version =0;
Query OK, 1 row affected (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from tx;
+----+-----+---------+
| id | num | version |
+----+-----+---------+
|  1 | 300 |       0 |
+----+-----+---------+
1 row in set (0.00 sec)
````

然后用java跑个并发扣`num`的程序

````java
package com.sky.code.mysql;

import java.sql.*;
import java.util.concurrent.atomic.AtomicInteger;

public class DbCas {

    // JDBC 驱动名及数据库 URL
    static final String JDBC_DRIVER = "com.mysql.cj.jdbc.Driver";
    static final String DB_URL = "jdbc:mysql://192.168.5.129:3306/test";//这里的ip是你docker的host的ip

    // 数据库的用户名与密码，需要根据自己的设置
    static final String USER = "root";
    static final String PASS = "example";

    static AtomicInteger counter=new AtomicInteger(0);

    public static void consumer(){
        try {
            Connection conn = DriverManager.getConnection(DB_URL,USER,PASS);
            conn.setAutoCommit(false);

            Statement statement=conn.createStatement();


            while (true){

                String sql="select version from tx where id=1 and num<>0";
                ResultSet rs=statement.executeQuery(sql);

                if (rs.next()) {
                    int version = rs.getInt("version");

                    rs.close();

                    sql="update tx set num=num-1,version=version+1 where id=1 and version=" + version;

                    int record = statement.executeUpdate(sql);

                    if (record == 1) {
                        counter.getAndIncrement();
                    }

                    conn.commit();
                }
                else {
                    break;
                }
            }

            statement.close();
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args){
        try {
            Class.forName(JDBC_DRIVER);

            Thread[] threads=new Thread[10];

            for(int i=0;i<10;i++){
                Thread thread=new Thread(() -> consumer());
                thread.start();
                threads[i]=thread;
            }

            for(int i=0;i<10;i++){
                try {
                    threads[i].join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            System.out.printf("finally update %d records",counter.get());

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
````

运行结果为

````bash
finally update 300 records
````

数据库数据

````sql
mysql> select * from tx;
+----+-----+---------+
| id | num | version |
+----+-----+---------+
|  1 |   0 |     300 |
+----+-----+---------+
1 row in set (0.00 sec)
````

你会发现，只有300个成功更新的记录,数据库的记录也没有超扣。

所以利用这个，可以不用`select for update `等锁的操作。

# 再试下锁

接下来试试锁吧

第一个事务执行

````sql
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from tx;
+----+-----+---------+
| id | num | version |
+----+-----+---------+
|  1 | 300 |       1 |
+----+-----+---------+
1 row in set (0.00 sec)

mysql> update tx set num =400 where version=1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
````

这里没有提交事务

然后第二个终端，直接运行下面命令

````sql
mysql> update tx set num =400 where version=1;

````
你会发现卡在那里，过一会就会响应

````
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
````

所以这里你就看到是有锁了，是不是很神奇呢，其实隔离级别就是靠锁来实现的

# 总结

上面列子和代码可以很好的说明了，数据库会在适当的时候加锁，保证数据不会有问题，当然前提是隔离级别设置够高，mysql默认是可重复读，所以足够保证了，还有上面例子可以用来做扣库存的代码哦😄，java代码可以在[这里](https://github.com/ejunjsh/java-code/blob/master/basic/src/main/java/com/sky/code/mysql/DbCas.java)拿到。
