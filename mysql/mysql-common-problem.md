### MySQL常见问题

通常我们在使用 MySQL 的过程中，会遇到一些问题，本文档列出来一些常见的问题以及处理方法

#### 忘记 root 账户密码

如果忘记了 MySQL 的 root 密码，首先需要停止 mysql 服务

```
$ killall -TERM mysqld
```

然后手动启动 mysql 服务，启动的时候加上 `--skip-grant-tables` 参数，表示跳过授权表启动数据库

```
$ mysqld_safe --user=root --skip-grant-tables &
```

重新用 root 登录数据库

```
$ mysql -u root
```

重新修改 root 密码

```
mysql> use mysql
mysql> update user set password=password("new_pass") where user="root";
mysql> flush privileges;
```

MySQL5.7 版本修改 root 密码方法

```
mysql> update mysql.user set authentication_string=password('new_pass') where user='root' ;
mysql> flush privileges;
```

#### MySQL主从不同步问题

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

在 Slave 服务器上查看 slave 状态。如果显示 Slave_IO_Running: Yes 属正常，Slave_SQL_Running: No 表示不同步
   
```
mysql> show slave status\G
```
   
**强制忽略错误继续同步** (适用于主从库数据相差不大，或者要求数据可以不完全统一的情况，数据要求不严格的情况)

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

**重新做主从，完全同步** (适用于主从库数据相差较大，或者要求数据完全统一的情况)

   
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
mysql> change master to master_host = '192.168.1.100',
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

#### MySQL从库同步慢问题

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

如果 innodb_flush_log_at_trx_commit 设置为 0，log buffer 将每秒一次地写入 log file中，并且logfile的flush(刷到磁盘)操作同时进行。该模式下，在事务提交的时候，不会主动触发写入磁盘的操作。

如果 innodb_flush_log_at_trx_commit 设置为 1，每次事务提交时 MySQL 都会把 log buffer 的数据写入 log file，并且 flush (刷到磁盘)中去.

如果 innodb_flush_log_at_trx_commit 设置为 2，每次事务提交时 MySQL 都会把 log buffer 的数据写入 log file 但是 flush (刷到磁盘)操作并不会同时进行。该模式下, MySQL 会每秒执行一次 flush (刷到磁盘)操作。

-----------------------------------------------------------------------------------------

Author: typefo <typefo@qq.com> 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)
