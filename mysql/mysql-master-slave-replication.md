#+title: MySQL主从复制
#+author: typefo
#+email: typefo@qq.com
#+options: ^:nil \n:t creator:nil
#+language: zh-CN

#+html_head: <link href="http://cdn.bootcss.com/bootstrap/2.3.2/css/bootstrap.min.css" rel="stylesheet">
#+html_head: <style type="text/css">body{padding:15px;margin:0 auto;width:1024px;font-size:15px;line-height:24px}</style>
#+html_head: <style type="text/css">h1{font-size:28px} h2{font-size:24px} h3{font-size:18px} h4{font-size:16px}</style>
#+html_head: <style type="text/css">p{text-indent:10px}</style>

* MySQL主从复制配置
  
  - master 服务器 192.168.1.100
  - slave 服务器 192.168.1.102
    
  *master 服务器 /etc/my.cnf 配置文件*
  
  #+BEGIN_EXAMPLE
  [mysqld]
  server-id = 1                           # 服务器 ID 标识，同一局域网内不可与其他服务器重复
  log-bin = mysql-bin                     # 二进制日志名称，开启 bin-log
  binlog_format = mixed                   # binlog 日志格式，mixed 表示混合模式
  binlog-do-db = test                     # 指定哪些数据库需要写入 binlog 日志
  binlog_ignore_db = mysql                # 指定哪些数据库不写入 binlog 日志
  max_binlog_size = 500M                  # binlog 文件最大大小，超过大小会生成新文件
  #+END_EXAMPLE
  
  在 master 上重启 mysql 数据库
  
  #+BEGIN_EXAMPLE
  $ service mysqld restart
  #+END_EXAMPLE
  
  *slave 服务器 /etc/my.cnf 配置*
  
  #+BEGIN_EXAMPLE
  [mysqld]
  server-id = 2                           # 服务器 ID 标识，同一局域网内不可与其他服务器重复
  log-bin = mysql-bin                     # 开启 binlog 二进制日志，并指定日志文件名称
  binlog_format = mixed                   # binlog 日志格式，mixed 表示混合模式
  relay-log = slave-relay-bin             # 设置 relay 中继日志文件名
  relay-log-index = slave-relay-bin.index # 设置 relay 中继日志索引文件名
  replicate_do_db = test                  # 指定哪些数据库需要同步复制
  replicate_ignore_db = mysql             # 指定哪些数据库不需要同步
  max_binlog_size = 500M                  # binlog 文件最大大小，超过大小会生成新文件
  log-slave-updates = 1                   # 如果这台 slave 同时也是其他服务器的 master 则必须开启此选项
  #+END_EXAMPLE
  
  在 slave 上重启 mysql 数据库
  
  #+BEGIN_EXAMPLE
  $ service mysqld restart
  #+END_EXAMPLE
  
  在 master 服务器上创建主从复制账号
  
  #+BEGIN_EXAMPLE
  mysql> grant replication slave on *.* to 'repluser'@'192.168.1.102' identified by '123456';
  #+END_EXAMPLE
  
  在 master 上查看并记录下 binlog 日志文件名和偏移点
  
  #+BEGIN_EXAMPLE
  mysql> show master status;
  +------------------+----------+--------------+------------------+
  | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
  +------------------+----------+--------------+------------------+
  | mysql-bin.000001 |      259 |              |                  |
  +------------------+----------+--------------+------------------+
  #+END_EXAMPLE
  
  在 slave 从服务器上配置 master 同步复制信息
  
  #+BEGIN_EXAMPLE
  mysql> change master to master_host="192.168.1.100",
                          master_user="repluser",
                          master_password="123456",
                          master_log_file="mysql-bin.000001",
                          master_log_pos=259;
  #+END_EXAMPLE
  
  在 slave 服务器上启动 master 同步
  
  #+BEGIN_EXAMPLE
  mysql> start slave;
  #+END_EXAMPLE
  
  
  在 slave 上查看主从复制是否成功
  
  #+BEGIN_EXAMPLE
  mysql> show slave status\G
  #+END_EXAMPLE
  
  如果出现 Slave_IO_Running: Yes 和 Slave_SQL_Running: Yes 表示主从同步正常
  测试主从同步效果，在主节点上创建一个数据库 test 或一张表 table，然后在从节点上查看是否有 test 数据库或 table 表的创建。  
  
  *将 slave 从数据库设置为只读模式*
  
  在 slave 的 /etc/my.cnf 配置文件中添加下面的选项可以使从数据库变为只读模式
  
  
  #+BEGIN_EXAMPLE
  [mysqld]
  read-only
  #+END_EXAMPLE
  
  或者设置一个 read_only 全局变量为 1
  
  #+BEGIN_EXAMPLE
  mysql> set global read_only = 1;
  #+END_EXAMPLE
  
  read_only = 1 只读模式，可以限定普通用户进行数据修改的操作，但不会限定具有 super 权限的用户（如超级管理员 root 用户）的数据修改操作。在 MySQL 中设置 read_only = 1 后，普通的应用用户进行 insert、update、delete 等会产生数据变化的 DML 操作时，都会报出数据库处于只读模式不能发生数据变化的错误，但具有 super 权限的用户，例如在本地或远程通过 root 用户登录到数据库，还是可以进行数据变化的 DML 操作；为了确保所有用户，包括具有 super 权限的用户也不能进行读写操作，就需要执行给所有的表加读锁的命令
  
  #+BEGIN_EXAMPLE
  mysql> flush tables with read lock;
  #+END_EXAMPLE
  
  这样使用具有 super 权限的用户登录数据库，想要发生数据变化的操作时，也会提示表被锁定不能修改的报错。将 slave 数据库 read-only = 1 设置只读后，在 master 执行 
  
  #+BEGIN_EXAMPLE
  mysql> GRANT USAGE ON *.* TO 'user'@'localhost' IDENTIFIED BY'123456' WITH GRANT OPTION;
  #+END_EXAMPLE
  
  创建一个普通用户，然后用普通用户登录从库，执行操作会报错。切换到 root 用户后还是可以进行增删改查的。
  
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
* MySQL同步慢IO优化

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

  sync_binlog 默认为 0，类似操作系统的磁盘文件刷新同步机制，MySQL 不会同步到磁盘中去而是依赖操作系统来刷新 binary log。当 sync_binlog =N (N>0) ，MySQL 在每写 N 次二进制日志 binary log 时，会使用 fdatasync() 函数将它的写二进制日志 binary log 同步到磁盘中去。如果启用了 autocommit，那么每一个语句 statement 就会有一次写操作，否则每个事务对应一个写操作。而且 autocommit 默认为开启

  如果 innodb_flush_log_at_trx_commit 设置为 0，log buffer 将每秒一次地写入 log file 中，并且 log file 的 flush (刷到磁盘)操作同时进行。该模式下，在事务提交的时候，不会主动触发写入磁盘的操作。

  如果 innodb_flush_log_at_trx_commit 设置为 1，每次事务提交时 MySQL 都会把 log buffer 的数据写入 log file，并且 flush (刷到磁盘)中去.

  如果 innodb_flush_log_at_trx_commit 设置为 2，每次事务提交时 MySQL 都会把 log buffer 的数据写入 log file 但是 flush (刷到磁盘)操作并不会同时进行。该模式下, MySQL 会每秒执行一次 flush (刷到磁盘)操作。
* 手动复制同步数据库
  
  当需要 Master 与 Slave 数据库全量同步时，如果数据量很大，可采用直接手动复制数据库目录拷贝到 Slave 服务器的方式

  在 Master 上停止 MySQL 服务
  
  #+BEGIN_EXAMPLE
  $ service mysqld stop
  #+END_EXAMPLE
  
  在 Master 上拷贝并打包 MySQL 数据库目录
  
  #+BEGIN_EXAMPLE
  $ tar -czvf master-backup.tar.gz /var/lib/mysql
  #+END_EXAMPLE

  在 Master 上拷贝打包完 MySQL 数据库目录后，启动 Master 上的 MySQL 服务

  #+BEGIN_EXAMPLE
  $ service mysqld start
  #+END_EXAMPLE

  在 Master 服务器上创建 Slave 主从复制账号

  #+BEGIN_EXAMPLE
  mysql> grant replication slave on *.* to 'repluser'@'192.168.1.102' identified by '123456';
  #+END_EXAMPLE

  然后在 Master 上查询并记录下当前 binlog 文件名及偏移位置
  
  #+BEGIN_EXAMPLE
  mysql> show master status;
  +------------------+----------+--------------+------------------+
  | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
  +------------------+----------+--------------+------------------+
  | mysql-bin.000001 |  25935678|              |                  |
  +------------------+----------+--------------+------------------+
  #+END_EXAMPLE

  把在 Master 上拷贝打包的 MySQL 数据库文件复制到 slave 从服务器上
  
  #+BEGIN_EXAMPLE
  $ scp master-backup.tar.gz root@192.168.1.102:/root
  #+END_EXAMPLE
  
  在 slave 从服务器上停止 MySQL 服务，清空 Slave 上的 MySQL 数据库目录，然后将 master-backup.tar.gz 解压覆盖到 Slave 上的 MySQL 数据库目录下
  
  #+BEGIN_EXAMPLE
  $ server msyqld stop
  $ rm -rf /var/lib/mysql/*
  $ tar -zxvf master-backup.tar.gz -C /var/lib/mysql
  #+END_EXAMPLE
  
  在 Slave 上启动 MySQL 服务

  #+BEGIN_EXAMPLE
  # service mysqld start
  #+END_EXAMPLE
  
  在 Slave 上重新设置 Master 主从同步信息
  
  #+BEGIN_EXAMPLE
  mysql> change master to master_host="192.168.1.100",
                          master_user="repluser",
                          master_password="123456",
                          master_log_file="mysql-bin.000001",
                          master_log_pos=25935678;
  #+END_EXAMPLE
  
  在 Slave 从服务器上启动 Master 主从同步复制
  
  #+BEGIN_EXAMPLE
  mysql> start slave;
  #+END_EXAMPLE
  
  在 Slave 从服务器上查看 Master 主从同步状态
  
  #+BEGIN_EXAMPLE
  mysql> show slave status\G
  #+END_EXAMPLE
* binlog-do-db注意事项

  binlog-do-db 表示哪些数据库需要记录 binlog 日志。

  *当 binlog-format 为 statement 时*

  在使用了 use dbname 的情况下，如果 dbname 没有在 binlog-do-db 里，数据库的 DDL 和 DML 操作都不会被记录到 binlog 日志，即使使用了类似 test.table 的方式也不会被记录到 binlog 日志

  *当 binlog-format 为 row 时*

  在使用了 use dbname 的情况下，如果 dbname 没有在 binlog-do-db 里，数据库的 DDL 操作不会被记录到 binlog 日志。在使用或不使用 use dbname时， DML 操作都被记录到 binlog 日志

  *当 binlog-format 为 mixed 时*

  在使用了 =use db= 的情况下，并且 db 在 binlog-do-db 里面 ，下列操作都会被记录 binlog 日志。

  #+BEGIN_EXAMPLE
  mysql> use db;
  mysql> create table table1(id int, value varchar(8));
  mysql> insert into table1 values(1, '11111111');
  mysql> update table1 set value = 'aaaaaaa' where id = 1;
  mysql> delete from table1 where id = 1;
  mysql> drop table table1;
  #+END_EXAMPLE

  在使用了 =use db= 的情况下，并且 db 不在 binlog-do-db 里面，下列操作都不会被记录 binlog 日志。

  #+BEGIN_EXAMPLE
  mysql> use db;
  mysql> create table table1(id int, value varchar(8));
  mysql> insert into table1 values(1, '11111111');
  mysql> update table1 set value = 'aaaaaaa' where id = 1;
  mysql> delete from table1 where id = 1;
  mysql> drop table table1;
  #+END_EXAMPLE

  在没有使用 =use db= 的情况下，并且 db 在 binlog-do-db 里面 ，下列操作都会被记录 binlog 日志。

  #+BEGIN_EXAMPLE
  mysql> create table test.table1(id int, value varchar(8));
  mysql> insert into test.table1 values(1, '11111111');
  mysql> update test.table1 set value = 'aaaaaaa' where id = 1;
  mysql> delete from test.table1 where id = 1;
  mysql> drop table test.table1;
  #+END_EXAMPLE

  在没有使用 =use db= 的情况下，并且 db 不在 binlog-do-db 里面 ，下列操作都会被记录 binlog 日志。

  #+BEGIN_EXAMPLE
  mysql> create table db.table1(id int, value varchar(8));
  mysql> insert into db.table1 values(1, '11111111');
  mysql> update db.table1 set value = 'aaaaaaa' where id = 1;
  mysql> delete from db.table1 where id = 1;
  mysql> drop table db.table1;
  #+END_EXAMPLE

* replicate_do_db注意事项

  replicate_do_db 参数是在 slave 上配置，指定 slave 要复制哪些数据库

  *当 binlog-format 为 statement 时*

  update 操作将不会被复制到 slave 上。

  *当 binlog-format 为 row 时*

  update 操作会复制到 slave 上。

  *当 binlog-format 为 mixed 时*

  在使用 use db 的情况下，并且 db 在 replicate_do_db 里面，下列操作都会被同步到 slave 从服务器

  #+BEGIN_EXAMPLE
  mysql> use db;
  mysql> create table table1(id int, value varchar(8));
  mysql> insert into table1 values(1, '1111111');
  mysql> update table1 set value = 'aaaaaaa' where id = 1;
  mysql> delete from table1 where id = 1;
  msyql. drop table table1;
  #+END_EXAMPLE

  在使用 use db 的情况下，并且 db 不在 replicate_do_db 里面，下列操作都不会被同步到 slave 从服务器

  #+BEGIN_EXAMPLE
  mysql> use db;
  mysql> create table table1(id int, value varchar(8));
  mysql> insert into table1 values(1, '1111111');
  mysql> update table1 set value = 'aaaaaaa' where id = 1;
  mysql> delete from table1 where id = 1;
  msyql. drop table table1;
  #+END_EXAMPLE

  在没有使用 use db 的情况下，并且 db 在 replicate_do_db 里面，下列操作都不会被同步到 slave 从服务器

  #+BEGIN_EXAMPLE
  mysql> create table db.table1(id int, value varchar(8));
  mysql> insert into db.table1 values(1, '1111111');
  mysql> update db.table1 set value = 'aaaaaaa' where id = 1;
  mysql> delete from db.table1 where id = 1;
  msyql. drop table db.table1;
  #+END_EXAMPLE

  在没有使用 use db 的情况下，并且 db 不在 replicate_do_db 里面，下列操作都不会被同步到 slave 从服务器

  #+BEGIN_EXAMPLE
  mysql> create table db.table1(id int, value varchar(8));
  mysql> insert into db.table1 values(1, '1111111');
  mysql> update db.table1 set value = 'aaaaaaa' where id = 1;
  mysql> delete from db.table1 where id = 1;
  msyql. drop table db.table1;
  #+END_EXAMPLE
