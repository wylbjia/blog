# MySQL 性能监控与分析

平时在做 MySQL 的运维过程中，总会遇到各种故障和性能问题，本文档简单的介绍一些常见性能监控分析方法

## 查看所有会话连接信息
   
查看当前系统中所有会话连接信息

```
MySQL> show full processlist;
```

`Id` 表示当前会话 ID 标识，`User` 表示连接登录的用户， `Host` 远程主机地址， `db` 当前连接的数据库， `Time` 会话连接时长， `Info` 正在执行的命令， `Progress` 表示 SQL 耗时时间。当发现某一个会话耗时太长，可以使用 kill 命令 + 会话ID，来强制终止会话。

```
MySQL> kill 12
```

## 使用慢查询日志记录耗时操作

数据库在使用过程中，有时会出现一些 SQL 语句执行太慢，启用 MySQL 的慢查询日志，可以记录所有耗时的 SQL 查询，以便做性能分析和优化。

```
MySQL> show variables like 'slow_query_log';
```

上面查询 slow_query_log 值为 OFF 就表示没有启用慢查询日志。可以在 MySQL 运行期间动态开启慢查询日志记录耗时 SQL 查询，如下

```
MySQL> set global slow_query_log = ON;
MySQL> set global long_query_time = 1;
```

使用上面的命令只对当前数据库生效。也可以在 my.cnf 配置文件中加入下列选项，使其永久生效

```
[mysqld]
slow_query_log
long_query_time = 3
log_queries_not_using_indexes
slow_query_log_file = /var/lib/mysql/slowquery.log
```

slow_query_log 表示开启慢查询日志，long_query_time 表示执行时间超过 3 秒的 SQL 查询将被记录。log_queries_not_using_indexes 表示没有用到索引的 SQL 查询也将被记录。slow_query_log_file 表示慢查询日志文件

## 使用 mysqldumpslow 分析慢查询日志
   
mysqldumpslow 工具可以自动分析和排序 MySQL 的慢查询日志文件，基本使用格式如下
   
```
mysqldumpslow [option] files
```
   
输出执行最多次的前 10 个 SQL 查询

```
$ mysqldumpslow -s c -t 10 /var/lib/mysql/slowquery.log
```

输出查询时间最长的前 10 个 SQL 查询
   
```
$ mysqldumpslow -s t -t 10 /var/lib/mysql/slowquery.log
```
 
输出返回数据记录数最多的前 10 个 SQL 查询

```
$ mysqldumpslow -s r -t 10 /var/lib/mysql/slowquery.log
```

输出按查询时间排序里面包含 count 关键字语句的 SQL 查询
   
```
$ mysqldumpslow -s t -t 10 -g 'count' /var/lib/mysql/slowquery.log
```

## 查看当前线程运行数

使用下面这个命令可以查看当前 MySQL 服务器的线程数量

```
MySQL> show global status like '%threads_runn%';
```

## 查看 MySQL 表锁争用情况

查看 table_locks_waited 和 table_locks_immediate 两个状态值，如果  table_locks_waited 的值比较高，说明有严重的表锁争用

```
mysql> show status like 'table%';
```

## 查看 InnoDB 表锁争用情况

查看 MySQL 的 InnoDB 表的锁争用情况，如果 Innodb_row_lock_waits 和 Innodb_row_lock_time_avg 值比较高，说明有严重锁争用
   
```
mysql> show status like 'innodb_row_lock%';
```

-------------------------------------------------------------------------------

By typefo typefo@qq.com Update: 2017-04-20 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)
