# MySQL 主从复制架构

MySQL 的主从复制架构采用的是主 Master 下面对接一个或多个 Slave 从服务器进行主从同步复制。主要用于读写分离，主库备份，读多写少的应用场景。同时主从复制的方案也有其他的一些主从组合方式。本文档只讲解单一的一主一从同步复制方案，也就是 Master 主服务器提供读写操作，Slave 从服务器只提供读操作。

## MySQL 主从复制配置

- 操作系统 CentOS 6.9 环境
- master 服务器 192.168.1.101
- slave 服务器 192.168.1.102

**master 服务器配置**

在 Master 上编辑  /etc/my.cnf 配置文件

```
[mysqld]
server-id = 1                           # 服务器 ID 标识，同一局域网内不可与其他服务器重复
log-bin = mysql-bin                     # 开启 bin-log 日志，mysql-bin 为日志文件前缀
binlog_format = mixed                   # binlog 日志格式，mixed 表示混合模式
binlog-do-db = test                     # 指定哪些数据库需要写入 binlog 日志
binlog_ignore_db = mysql                # 指定哪些数据库不写入 binlog 日志
max_binlog_size = 500M                  # binlog 文件最大大小，超过该大小会生成新的日志文件
```
  
在 master 上重启 mysql 数据库
  
```
$ service mysqld restart
```

在 master 服务器上创建主从复制账号
  
```
mysql> grant replication slave on *.* to 'repluser'@'192.168.1.102' identified by '123456';
```

**slave 服务器配置**

在 Slave 上编辑  /etc/my.cnf 配置文件

```
[mysqld]
server-id = 2                           # 服务器 ID 标识，同一局域网内不可与其他服务器重复
log-bin = mysql-bin                     # 开启 bin-log 日志，mysql-bin 为日志文件前缀
binlog_format = mixed                   # binlog 日志格式，mixed 表示混合模式
relay-log = slave-relay-bin             # 设置 relay 中继日志文件名
relay-log-index = slave-relay-bin.index # 设置 relay 中继日志索引文件名
replicate_do_db = test                  # 指定哪些数据库需要同步复制
replicate_ignore_db = mysql             # 指定哪些数据库不需要同步
max_binlog_size = 500M                  # binlog 文件最大大小，超过该大小会生成新的日志文件
log-slave-updates = 1                   # 如果这台 slave 同时也是其他服务器的 master 则必须开启此选项
```
  
在 slave 上重启 mysql 数据库
  
```
$ service mysqld restart
```
  
在 master 上查看并记录下 binlog 日志文件名和偏移点
  
```
mysql> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      259 |              |                  |
+------------------+----------+--------------+------------------+
```
  
在 slave 从服务器上配置 master 同步复制信息
  
```
mysql> change master to master_host="192.168.1.101",
                        master_user="repluser",
                        master_password="123456",
                        master_log_file="mysql-bin.000001",
                        master_log_pos=259;
```
  
在 slave 服务器上启动 master 同步
  
```
mysql> start slave;
```
  
  
在 slave 上查看主从复制是否成功
  
```
mysql> show slave status\G
```
  
如果出现 Slave_IO_Running: Yes 和 Slave_SQL_Running: Yes 表示主从同步正常，测试主从同步效果，在主节点上创建一个数据库 test 或一张表 table，然后在从节点上查看是否有 test 数据库或 table 表的创建。  
  
**将 slave 从数据库设置为只读模式**
  
在 slave 的 /etc/my.cnf 配置文件中添加下面的选项可以使从数据库变为只读模式
  
```
[mysqld]
read-only
```

或者设置一个 read_only 全局变量为 1
  
```
mysql> set global read_only = 1;
```
  
read_only = 1 只读模式，可以限定普通用户进行数据修改的操作，但不会限定具有 super 权限的用户（如超级管理员 root 用户）的数据修改操作。在 MySQL 中设置 read_only = 1 后，普通的应用用户进行 insert、update、delete 等会产生数据变化的 DML 操作时，都会报出数据库处于只读模式不能发生数据变化的错误，但具有 super 权限的用户，例如在本地或远程通过 root 用户登录到数据库，还是可以进行数据变化的 DML 操作；为了确保所有用户，包括具有 super 权限的用户也不能进行读写操作，就需要执行给所有的表加读锁的命令
  
```
mysql> flush tables with read lock;
```
  
这样使用具有 super 权限的用户登录数据库，想要发生数据变化的操作时，也会提示表被锁定不能修改的报错。将 slave 数据库 read-only = 1 设置只读后，在 master 执行 
  
```
mysql> GRANT USAGE ON *.* TO 'user'@'localhost' IDENTIFIED BY'123456' WITH GRANT OPTION;
```
  
创建一个普通用户，然后用普通用户登录从库，执行操作会报错。切换到 root 用户后还是可以进行增删改查的。
  
## MySQL主从不同步问题
   

在 Master 服务器上查看当前连接会话信息，查看进程 Sleep 是否太多
   
```
mysql> show processlist;
```
   
查看 master 状态
   
```
mysql> show master status;
+-------------------+----------+--------------+-------------------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB              |
+-------------------+----------+--------------+-------------------------------+
| mysqld-bin.000001 |     3260 |              | mysql,test,information_schema |
+-------------------+----------+--------------+-------------------------------+
```

在 Slave 服务器上查看 slave 状态
   
```
mysql> show slave status\G
```
   
输出结果中 Slave_IO_Running: Yes 属正常，Slave_SQL_Running: No 表示不同步
   
   
**忽略错误后，继续同步**
   
该方法适用于主从库数据相差不大，或者要求数据可以不完全统一的情况，数据要求不严格的情况

1. 在 slave 从服务器上停止 slave 同步

```
mysql> stop slave;
```
   
2. 然后设置全局变量 sql_slave_skip_counter = 1 表示跳过一步错误，后面的数字可变
   
```
mysql> set global sql_slave_skip_counter = 1;
```
   
3. 然后启动 slave 同步

```
mysql> start slave;
```
   

4. 再查看当前同步状态

```
mysql> show slave status\G
```

   
**重新做主从，完全同步**
   
该方法适用于主从库数据相差较大，或者要求数据完全统一的情况
   

1. 在 master 上进行锁表，让其变成只读模式，防止数据写入
   
```
mysql> flush tables with read lock;
```
   
2. 将 master 上的所有数据库进行导出
   
```
$ mysqldump -u root -p > master-backuup.sql
```
   
这里注意一点：数据库备份一定要定期进行，确保数据万无一失
   
3. 查看 master 状态
   
```
mysql> show master status;
+-------------------+----------+--------------+-------------------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB              |
+-------------------+----------+--------------+-------------------------------+
| mysqld-bin.000001 |     3260 |              | mysql,test,information_schema |
+-------------------+----------+--------------+-------------------------------+
```

4. 把 master 备份文件传到 slave 从库机器，进行数据恢复
   
```
$ scp master-backup.sql root@192.168.1.102:/root
```
   
5. 在 slave 机器上停止从库的同步状态
   
```
mysql> stop slave;
```
   
6. 然后在 slave 从库机器上执行导入 master-backup.sql 数据备份
   
```
mysql> source /root/master-backup.sql
```
   
7. 设置 slave 从库同步，注意该处的同步点，就是 master 主库 'show master status' 信息里的 File 和 Position 两个字段的值
   
```
mysql> change master to master_host = '192.168.1.101',
                        master_user = 'repluser',
                        master_port=3306,
                        master_password='123456',
                        master_log_file = 'mysqld-bin.000001',
                        master_log_pos=3260;
```
   
8. 重新开启从同步
   
```
mysql> start slave;
```
   
9. 查看同步状态
   
```
mysql> show slave status\G
```
   
确认 Slave_IO_Running: Yes 和 Slave_SQL_Running: Yes 都为 'Yes' 就表示主从同步正常

## MySQL 同步慢 IO 优化

检查 slave 服务器 io thread 和 sql thread 状态

```
mysql> show slave status;
```

slave 服务器正常空闲状态是下面这样

```
Slave_IO_State: Waiting for master to send event
Slave_IO_Running: Yes
Seconds_Behind_Master: 0
```

slave 服务器 sql thread 慢的表现为

```
Seconds_Behind_Master 值越来越大
```
  
slave 服务器 io thread 慢的表现为 

```
Slave_IO_State: Waiting for master to send event
Seconds_Behind_Master: 313
Slave_SQL_Running_State: Reading event from the relay log
```

当服 slave 务器磁盘 IO 速度比较低的时候会导致 slave 服务器同步速度跟不上其他从服务器，需要调整相关 MySQL 参数

```
mysql> set global sync_binlog = 20;
mysql> set global innodb_flush_log_at_trx_commit = 2;
```

sync_binlog 默认为 0，类似操作系统的磁盘文件刷新同步机制，MySQL 不会同步到磁盘中去而是依赖操作系统来刷新 binary log。当 sync_binlog =N (N>0) ，MySQL 在每写 N 次二进制日志 binary log 时，会使用 fdatasync() 函数将它的写二进制日志 binary log 同步到磁盘中去。如果启用了 autocommit，那么每一个语句 statement 就会有一次写操作，否则每个事务对应一个写操作。而且 autocommit 默认为开启

如果 innodb_flush_log_at_trx_commit 设置为 0，log buffer 将每秒一次地写入 log file 中，并且 log file 的 flush (刷到磁盘)操作同时进行。该模式下，在事务提交的时候，不会主动触发写入磁盘的操作。

如果 innodb_flush_log_at_trx_commit 设置为 1，每次事务提交时 MySQL 都会把 log buffer 的数据写入 log file，并且 flush (刷到磁盘)中去.

如果 innodb_flush_log_at_trx_commit 设置为 2，每次事务提交时 MySQL 都会把 log buffer 的数据写入 log file 但是 flush (刷到磁盘)操作并不会同时进行。该模式下, MySQL 会每秒执行一次 flush (刷到磁盘)操作。

## 手动复制同步数据库
  
当需要 Master 与 Slave 数据库全量同步时，如果数据量很大，可采用直接手动复制数据库目录拷贝到 Slave 服务器的方式

在 Master 上停止 MySQL 服务
  
```
$ service mysqld stop
```
  
在 Master 上拷贝并打包 MySQL 数据库目录
  
```
$ tar -czvf master-backup.tar.gz /var/lib/mysql
```

在 Master 上拷贝打包完 MySQL 数据库目录后，启动 Master 上的 MySQL 服务

```
$ service mysqld start
```

在 Master 服务器上创建 Slave 主从复制账号

```
mysql> grant replication slave on *.* to 'repluser'@'192.168.1.102' identified by '123456';
```

然后在 Master 上查询并记录下当前 binlog 文件名及偏移位置
  
```
mysql> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |  25935678|              |                  |
+------------------+----------+--------------+------------------+
```

把在 Master 上拷贝打包的 MySQL 数据库文件复制到 slave 从服务器上
  
```
$ scp master-backup.tar.gz root@192.168.1.102:/root
```
  
在 slave 从服务器上停止 MySQL 服务，清空 Slave 上的 MySQL 数据库目录，然后将 master-backup.tar.gz 解压覆盖到 Slave 上的 MySQL 数据库目录下
  
```
$ server msyqld stop
$ rm -rf /var/lib/mysql/*
$ tar -zxvf master-backup.tar.gz -C /var/lib/mysql
```
  
在 Slave 上启动 MySQL 服务

```
# service mysqld start
```
  
在 Slave 上重新设置 Master 主从同步信息
  
```
mysql> change master to master_host="192.168.1.101",
                        master_user="repluser",
                        master_password="123456",
                        master_log_file="mysql-bin.000001",
                        master_log_pos=25935678;
```
  
在 Slave 从服务器上启动 Master 主从同步复制
  
```
mysql> start slave;
```
  
在 Slave 从服务器上查看 Master 主从同步状态
  
```
mysql> show slave status\G
```

## binlog-do-db 注意事项

binlog-do-db 表示哪些数据库需要记录 binlog 日志。

- 当 binlog-format 为 statement 时

在使用了 `use db` 的情况下，如果 dbname 没有在 binlog-do-db 里，数据库的 DDL 和 DML 操作都不会被记录到 binlog 日志，即使使用了类似 test.table 的方式也不会被记录到 binlog 日志

- 当 binlog-format 为 row 时

在使用了 `use db` 的情况下，如果 dbname 没有在 binlog-do-db 里，数据库的 DDL 操作不会被记录到 binlog 日志。在使用或不使用 use dbname时， DML 操作都被记录到 binlog 日志

- 当 binlog-format 为 mixed 时

在使用了 `use db` 的情况下，并且 db 在 binlog-do-db 里面 ，下列操作都会被记录 binlog 日志。

```
mysql> use db;
mysql> create table table1(id int, value varchar(8));
mysql> insert into table1 values(1, '11111111');
mysql> update table1 set value = 'aaaaaaa' where id = 1;
mysql> delete from table1 where id = 1;
mysql> drop table table1;
```

在使用了 `use db` 的情况下，并且 db 不在 binlog-do-db 里面，下列操作都不会被记录 binlog 日志。

```
mysql> use db;
mysql> create table table1(id int, value varchar(8));
mysql> insert into table1 values(1, '11111111');
mysql> update table1 set value = 'aaaaaaa' where id = 1;
mysql> delete from table1 where id = 1;
mysql> drop table table1;
```

在没有使用 `use db` 的情况下，并且 db 在 binlog-do-db 里面 ，下列操作都会被记录 binlog 日志。

```
mysql> create table test.table1(id int, value varchar(8));
mysql> insert into test.table1 values(1, '11111111');
mysql> update test.table1 set value = 'aaaaaaa' where id = 1;
mysql> delete from test.table1 where id = 1;
mysql> drop table test.table1;
```

在没有使用 `use db` 的情况下，并且 db 不在 binlog-do-db 里面 ，下列操作都会被记录 binlog 日志。

```
mysql> create table db.table1(id int, value varchar(8));
mysql> insert into db.table1 values(1, '11111111');
mysql> update db.table1 set value = 'aaaaaaa' where id = 1;
mysql> delete from db.table1 where id = 1;
mysql> drop table db.table1;
```

## replicate_do_db 注意事项

replicate_do_db 参数是在 slave 上配置，指定 slave 要复制哪些数据库

- 当 binlog-format 为 statement 时

update 操作将不会被复制到 slave 上。

- 当 binlog-format 为 row 时

update 操作会复制到 slave 上。

- 当 binlog-format 为 mixed 时

在使用 `use db` 的情况下，并且 db 在 replicate_do_db 里面，下列操作都会被同步到 slave 从服务器

```
mysql> use db;
mysql> create table table1(id int, value varchar(8));
mysql> insert into table1 values(1, '1111111');
mysql> update table1 set value = 'aaaaaaa' where id = 1;
mysql> delete from table1 where id = 1;
msyql. drop table table1;
```

在使用 `use db` 的情况下，并且 db 不在 replicate_do_db 里面，下列操作都不会被同步到 slave 从服务器

```
mysql> use db;
mysql> create table table1(id int, value varchar(8));
mysql> insert into table1 values(1, '1111111');
mysql> update table1 set value = 'aaaaaaa' where id = 1;
mysql> delete from table1 where id = 1;
msyql. drop table table1;
```

在没有使用 `use db` 的情况下，并且 db 在 replicate_do_db 里面，下列操作都不会被同步到 slave 从服务器

```
mysql> create table db.table1(id int, value varchar(8));
mysql> insert into db.table1 values(1, '1111111');
mysql> update db.table1 set value = 'aaaaaaa' where id = 1;
mysql> delete from db.table1 where id = 1;
msyql. drop table db.table1;
```

在没有使用 `use db` 的情况下，并且 db 不在 replicate_do_db 里面，下列操作都不会被同步到 slave 从服务器

```
mysql> create table db.table1(id int, value varchar(8));
mysql> insert into db.table1 values(1, '1111111');
mysql> update db.table1 set value = 'aaaaaaa' where id = 1;
mysql> delete from db.table1 where id = 1;
msyql. drop table db.table1;
```

------------------------------------------------------------------------------

By typefo typefo@qq.com Update: 2017-04-24 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)
