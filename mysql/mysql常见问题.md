#+title: MySQL常见问题
#+author: typefo
#+email: typefo@qq.com
#+options: ^:nil \n:t creator:nil
#+language: zh-CN

#+html_head: <link href="http://cdn.bootcss.com/bootstrap/2.3.2/css/bootstrap.min.css" rel="stylesheet">
#+html_head: <style type="text/css">body{padding:15px;margin:0 auto;width:1024px;font-size:15px;line-height:24px}</style>
#+html_head: <style type="text/css">h1{font-size:28px} h2{font-size:24px} h3{font-size:18px} h4{font-size:16px}</style>
#+html_head: <style type="text/css">p{text-indent:10px}</style>

* 忘记 root 账户密码

  停止 mysql 服务器

  #+BEGIN_EXAMPLE
  $ killall -TERM mysqld
  #+END_EXAMPLE

  跳过 mysql 授权表启动数据库

  #+BEGIN_EXAMPLE
  $ mysqld_safe --user=root --skip-grant-tables &
  #+END_EXAMPLE

  重新用 root 登录数据库

  #+BEGIN_EXAMPLE
  $ mysql -u root
  #+END_EXAMPLE

  重新修改 root 密码

  #+BEGIN_EXAMPLE
  mysql> use mysql
  mysql> update user set password=password("new_pass") where user="root";
  mysql> flush privileges;
  #+END_EXAMPLE

  MySQL5.7 版本修改 root 密码方法

  #+BEGIN_EXAMPLE
  mysql> update mysql.user set authentication_string=password('new_pass') where user='root' ;
  mysql> flush privileges;
  #+END_EXAMPLE

* MySQL主从不同步问题
   

   在 Master 服务器上查看当前连接会话信息，查看进程 Sleep 是否太多
   
   #+BEGIN_EXAMPLE
   mysql> show processlist;
   #+END_EXAMPLE
   
   查看 master 状态
   
   #+BEGIN_EXAMPLE
   mysql> show master status;
   +-------------------+----------+--------------+-------------------------------+
   | File              | Position | Binlog_Do_DB | Binlog_Ignore_DB              |
   +-------------------+----------+--------------+-------------------------------+
   | mysqld-bin.000001 |     3260 |              | mysql,test,information_schema |
   +-------------------+----------+--------------+-------------------------------+
   #+END_EXAMPLE

   在 Slave 服务器上查看 slave 状态
   
   #+BEGIN_EXAMPLE
   mysql> show slave status\G
   #+END_EXAMPLE
   
   输出结果中 Slave_IO_Running: Yes 属正常，Slave_SQL_Running: No 表示不同步
   
   
   *忽略错误后，继续同步*
   
   该方法适用于主从库数据相差不大，或者要求数据可以不完全统一的情况，数据要求不严格的情况

   1.在 slave 从服务器上停止 slave 同步

   #+BEGIN_EXAMPLE
   mysql> stop slave;
   #+END_EXAMPLE
   
   2.然后设置全局变量 sql_slave_skip_counter = 1 表示跳过一步错误，后面的数字可变
   
   #+BEGIN_EXAMPLE
   mysql> set global sql_slave_skip_counter = 1;
   #+END_EXAMPLE
   
   3.然后启动 slave 同步

   #+BEGIN_EXAMPLE
   mysql> start slave;
   #+END_EXAMPLE
   

   4.再查看当前同步状态

   #+BEGIN_EXAMPLE
   mysql> show slave status\G
   #+END_EXAMPLE

   
   *重新做主从，完全同步*
   
   该方法适用于主从库数据相差较大，或者要求数据完全统一的情况
   

   1.在 master 上进行锁表，让其变成只读模式，防止数据写入
   
   #+BEGIN_EXAMPLE
   mysql> flush tables with read lock;
   #+END_EXAMPLE
   
   2.将 master 上的所有数据库进行导出
   
   #+BEGIN_EXAMPLE
   $ mysqldump -u root -p > master-backuup.sql
   #+END_EXAMPLE
   
   这里注意一点：数据库备份一定要定期进行，确保数据万无一失
   
   3.查看 master 状态
   
   #+BEGIN_EXAMPLE
   mysql> show master status;
   +-------------------+----------+--------------+-------------------------------+
   | File              | Position | Binlog_Do_DB | Binlog_Ignore_DB              |
   +-------------------+----------+--------------+-------------------------------+
   | mysqld-bin.000001 |     3260 |              | mysql,test,information_schema |
   +-------------------+----------+--------------+-------------------------------+
   #+END_EXAMPLE

   4.把 master 备份文件传到 slave 从库机器，进行数据恢复
   
   #+BEGIN_EXAMPLE
   $ scp master-backup.sql root@192.168.1.102:/root
   #+END_EXAMPLE
   
   5.在 slave 机器上停止从库的同步状态
   
   #+BEGIN_EXAMPLE
   mysql> stop slave;
   #+END_EXAMPLE
   
   6.然后在 slave 从库机器上执行导入 master-backup.sql 数据备份
   
   #+BEGIN_EXAMPLE
   mysql> source /root/master-backup.sql
   #+END_EXAMPLE
   
   7.设置 slave 从库同步，注意该处的同步点，就是 master 主库 'show master status' 信息里的 File 和 Position 两个字段的值
   
   #+BEGIN_EXAMPLE
   mysql> change master to master_host = '192.168.1.100',
                           master_user = 'repluser',
                           master_port=3306,
                           master_password='123456',
                           master_log_file = 'mysqld-bin.000001',
                           master_log_pos=3260;
   #+END_EXAMPLE
   
   8.重新开启从同步
   
   #+BEGIN_EXAMPLE
   mysql> start slave;
   #+END_EXAMPLE
   
   9.查看同步状态
   
   #+BEGIN_EXAMPLE
   mysql> show slave status\G
   #+END_EXAMPLE
   
   确认 Slave_IO_Running: Yes 和 Slave_SQL_Running: Yes 都为 'Yes' 就表示主从同步正常
* MySQL从库同步慢问题

  检查 slave 服务器 io thread 和 sql thread 状态

  #+BEGIN_EXAMPLE
  mysql> show slave status;
  #+END_EXAMPLE

  slave 服务器正常空闲状态是下面这样

  #+BEGIN_EXAMPLE
  Slave_IO_State: Waiting for master to send event
  Slave_IO_Running: Yes
  Seconds_Behind_Master: 0
  #+END_EXAMPLE

  slave 服务器 sql thread 慢的表现为

  #+BEGIN_EXAMPLE
  Seconds_Behind_Master 值越来越大
  #+END_EXAMPLE
  
  slave 服务器 io thread 慢的表现为 

  #+BEGIN_EXAMPLE
  Slave_IO_State: Waiting for master to send event
  Seconds_Behind_Master: 313
  Slave_SQL_Running_State: Reading event from the relay log
  #+END_EXAMPLE

  当服 slave 务器磁盘 IO 速度比较低的时候会导致 slave 服务器同步速度跟不上其他从服务器，需要调整相关 MySQL 参数

  #+BEGIN_EXAMPLE
  mysql> set global sync_binlog = 20;
  mysql> set global innodb_flush_log_at_trx_commit = 2;
  #+END_EXAMPLE

  sync_binlog 默认为 0，类似操作系统的磁盘文件刷新同步机制，MySQL 不会同步到磁盘中去而是依赖操作系统来刷新 binary log。当 sync_binlog =N (N>0) ，MySQL 在每写 N 次二进制日志 binary log 时，会使用 fdatasync() 函数将它的写二进制日志 binary log 同步到磁盘中去。如果启用了 autocommit，那么每一个语句 statement 就会有一次写操作，否则每个事务对应一个写操作。
而且 autocommit 默认为开启

  如果 innodb_flush_log_at_trx_commit 设置为 0，log buffer 将每秒一次地写入 log file 中，并且 log file 的 flush (刷到磁盘)操作同时进行。该模式下，在事务提交的时候，不会主动触发写入磁盘的操作。

  如果 innodb_flush_log_at_trx_commit 设置为 1，每次事务提交时 MySQL 都会把 log buffer 的数据写入 log file，并且 flush (刷到磁盘)中去.

  如果 innodb_flush_log_at_trx_commit 设置为 2，每次事务提交时 MySQL 都会把 log buffer 的数据写入 log file 但是 flush (刷到磁盘)操作并不会同时进行。该模式下, MySQL 会每秒执行一次 flush (刷到磁盘)操作。
