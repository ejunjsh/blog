---
title: 开启mysql慢查询
date: 2017-09-20 21:47:38
tags: mysql
categories: mysql
---
# 简介
开启慢查询日志，可以让MySQL记录下查询超过指定时间的语句，通过定位分析性能的瓶颈，才能更好的优化数据库系统的性能。

# 参数说明
`slow_query_log` 慢查询开启状态
`slow_query_log_file` 慢查询日志存放的位置（这个目录需要MySQL的运行帐号的可写权限，一般设置为MySQL的数据存放目录）
`long_query_time` 查询超过多少秒才记录
<!-- more -->
# 设置步骤
1. 查看慢查询相关参数
    ````sql
    mysql> show variables like 'slow_query%';
    +---------------------------+----------------------------------+
    | Variable_name             | Value                            |
    +---------------------------+----------------------------------+
    | slow_query_log            | OFF                              |
    | slow_query_log_file       | /mysql/data/localhost-slow.log   |
    +---------------------------+----------------------------------+

    mysql> show variables like 'long_query_time';
    +-----------------+-----------+
    | Variable_name   | Value     |
    +-----------------+-----------+
    | long_query_time | 10.000000 |
    +-----------------+-----------+
    ````
2. 设置方法
    方法一：全局变量设置
    将 `slow_query_log` 全局变量设置为“ON”状态
    ````sql
    mysql> set global slow_query_log='ON'; 
    ````
    设置慢查询日志存放的位置
    ````sql
    mysql> set global slow_query_log_file='/usr/local/mysql/data/slow.log';
    ````
    查询超过1秒就记录
    ````sql
    mysql> set global long_query_time=1;
    ````
    方法二：配置文件设置
    修改配置文件my.cnf，在[mysqld]下的下方加入
    ````
    [mysqld]
    slow_query_log = ON
    slow_query_log_file = /usr/local/mysql/data/slow.log
    long_query_time = 1
    ````
3. 重启MySQL服务
    ````
    service mysqld restart
    ````
4. 查看设置后的参数
    ````sql
    mysql> show variables like 'slow_query%';
    +---------------------+--------------------------------+
    | Variable_name       | Value                          |
    +---------------------+--------------------------------+
    | slow_query_log      | ON                             |
    | slow_query_log_file | /usr/local/mysql/data/slow.log |
    +---------------------+--------------------------------+

    mysql> show variables like 'long_query_time';
    +-----------------+----------+
    | Variable_name   | Value    |
    +-----------------+----------+
    | long_query_time | 1.000000 |
    +-----------------+----------+
    ````

# 测试
1. 执行一条慢查询SQL语句
    ````sql
    mysql> select sleep(2);
    ````
2. 查看是否生成慢查询日志
    ````shell
    ls /usr/local/mysql/data/slow.log
    ````
    如果日志存在，MySQL开启慢查询设置成功！