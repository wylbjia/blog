#+title: MySQL常用命令
#+author: typefo
#+email: typefo@qq.com
#+options: ^:nil \n:t creator:nil
#+language: zh-CN

#+html_head: <link href="http://cdn.bootcss.com/bootstrap/2.3.2/css/bootstrap.min.css" rel="stylesheet">
#+html_head: <style type="text/css">body{padding:15px;margin:0 auto;width:1024px;font-size:15px;line-height:24px}</style>
#+html_head: <style type="text/css">h1{font-size:28px} h2{font-size:24px} h3{font-size:18px} h4{font-size:16px}</style>
#+html_head: <style type="text/css">p{text-indent:10px}</style>

* 账户管理
** 查询所有账户

   查询当前数据库所有用户账户

  #+BEGIN_EXAMPLE
  MySQL> select Host,User,Password from mysql.user;
  #+END_EXAMPLE

** 创建用户账户

   创建一个用户账户
  #+BEGIN_EXAMPLE
  MySQL> CREATE USER 'username'@'host' IDENTIFIED BY 'password';
  #+END_EXAMPLE

  =username= 表示账户名称， =host= 表示主机地址，百分号 '%' 表示任何主机，  =password= 表示用户密码

  创建一个 admin 用户账户，密码为 '123456'，并且可以从任何主机登录

   #+BEGIN_EXAMPLE
   MySQL> CREATE USER 'admin'@'%' IDENTIFIED BY '123456';
   #+END_EXAMPLE
   
** 用户账户授权

   GRANT 用户授权命令的基本格式

  #+BEGIN_EXAMPLE
  MySQL> GRANT privileges ON databasename.tablename TO 'username'@'host'
  #+END_EXAMPLE

  =privileges= 表示授权的操作，比如 SELECT, INSERT, UPDATE, DELETE 等等, ALL 代表所有操作
  =databasename= 表示所需要授权的数据库名称，可用星号 * 表示所有数据库
  =tablename= 表示所需要授权的表名称，可用星号 * 表示所有表
  =username= 授权的用户名
  =host= 授权的用户主机地址，百分号 '%' 表示可从任何主机连接登录

  注意:用以上命令授权的用户不能给其它用户授权,如果想让该用户可以授权,用以下命令:

  #+BEGIN_EXAMPLE
  MySQL> GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;
  #+END_EXAMPLE

  将数据库 test 的 select 权限赋予给 admin 用户

  #+BEGIN_EXAMPLE
  MySQL> GRANT SELECT ON test.* TO 'admin'@'192.168.1.130'
  #+END_EXAMPLE

  授予所有权限给 admin 用户

  #+BEGIN_EXAMPLE
  MySQL> GRANT ALL ON *.* TO 'admin'@'192.168.1.130'
  #+END_EXAMPLE

** 修改用户密码
  
  #+BEGIN_EXAMPLE
  MySQL> SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
  #+END_EXAMPLE

  =username= 需要修改的用户帐户， =host= 对应用户的允许连接登录主机， =newpassword= 新的密码

  修改当前登录用户自己的密码

  #+BEGIN_EXAMPLE
  MySQL> SET PASSWORD = PASSWORD("newpassword");
  #+END_EXAMPLE
  
  =newpasword= 新的密码

** 撤销用户权限

  #+BEGIN_EXAMPLE
  MySQL> REVOKE privilege ON databasename.tablename FROM 'username'@'host';
  #+END_EXAMPLE

  =privileges= 授权操作，比如 SELECT, INSERT, UPDATE, DELETE 等等, ALL 代表所有操作
  =databasename= 数据库名称，可用星号 * 表示所有数据库
  =tablename= 表名称，可用星号 * 表示所有表
  =username= 用户账户
  =host= 户主机地址，百分号 '%' 任何主机

  撤销用户 admin 对数据库 test 的 SELECT 权限

  #+BEGIN_EXAMPLE
  MySQL> REVOKE SELECT ON test.* FROM 'admin'@'192.168.1.130';
  #+END_EXAMPLE

** 删除用户账户

  删除一个用户账户的基本命令格式

  #+BEGIN_EXAMPLE
  MySQL> DROP USER 'username'@'host';
  #+END_EXAMPLE

  比如删除 admin 用户账户

  #+BEGIN_EXAMPLE
  MySQL> DROP USER 'admin'@'192.168.1.130';
  #+END_EXAMPLE

** 刷新权限表

   所有对用户的操作，最后都需要使用下面的命令刷新数据库权限表才能生效

   #+BEGIN_EXAMPLE
   MySQL> flush privileges;
   #+END_EXAMPLE

** 查看用户权限

   #+BEGIN_EXAMPLE
   MySQL> show grants for root;
   #+END_EXAMPLE

* 数据库管理
** 查看某个数据库是如何创建的

  #+BEGIN_EXAMPLE
  MySQL> show create database databasename;
  #+END_EXAMPLE

** 将 SQL 文件导入数据库

   #+BEGIN_EXAMPLE
   mysql> use database;
   mysql> source /root/backup.sql;
   #+END_EXAMPLE

* 数据表管理
** 查看某个表是如何创建的

   #+BEGIN_EXAMPLE
   MySQL> show create table tablename;
   #+END_EXAMPLE

** 查看一个表的列字段名称

   #+BEGIN_EXAMPLE
   MySQL> show columns from tablename;
   #+END_EXAMPLE

** mysql 锁表和解锁

   当需要对数据库做备份或者同步的时候，需要事先将表加锁，防止其他客户端写入数据，导致数据不一致

   #+BEGIN_EXAMPLE
   mysql> flush table with read lock;
   #+END_EXAMPLE

   将表解锁

   #+BEGIN_EXAMPLE
   mysql> unlock table;
   #+END_EXAMPLE

** 清空表数据但不删除表

   #+BEGIN_EXAMPLE
   mysql> delete from tablename;
   #+END_EXAMPLE

* 视图管理
* 索引管理
* 存储过程
** 查看所有存储过程

   #+BEGIN_EXAMPLE
   mysql> show procedure status;
   #+END_EXAMPLE

   或者

   #+BEGIN_EXAMPLE
   mysql> select * from mysql.proc;
   #+END_EXAMPLE

* 触发器管理
** 查看所有触发器

   #+BEGIN_EXAMPLE
   mysql> select * from information_schema.triggers;
   #+END_EXAMPLE

* 函数管理
** 查看所有函数

   #+BEGIN_EXAMPLE
   mysql> show function status;
   #+END_EXAMPLE

* 日志管理
** Binlog日志管理
   
   *删除所有 binlog 日志*

   #+BEGIN_EXAMPLE
   mysql> reset master;
   #+END_EXAMPLE

   *删除指定 binlog 日志之前的文件*

   #+BEGIN_EXAMPLE
   mysql> purge master logs to 'mysql-bin.000005';
   #+END_EXAMPLE

   上面的命令将会删除 mysql-bin.000005 之前生成的 binlog 日志文件，mysql-bin.000005 本身不会被删除

   *删除指定时间之前生成的 binlog 日志文件*

   #+BEGIN_EXAMPLE
   mysql> purge master logs before '2017-03-15 18:00:00';
   #+END_EXAMPLE

   上面的命令将会删除 '2017-03-15 18:00:00' 时间之前生成的 binlog 日志文件，这个时间点生成文件不会被删除
* 主从同步管理
** 刷新binlog日志

   刷新 binlog 日志会是服务器重新建立新的 binlog 日志文件

   #+BEGIN_EXAMPLE
   mysql> flush logs
   #+END_EXAMPLE

** 重置 Master

   重置 master 会清空所有 bin-log 日志，只保留 mysql-bin.000001 文件

   #+BEGIN_EXAMPLE
   mysql> reset master
   #+END_EXAMPLE

** 查看 binlog 记录事件

   #+BEGIN_EXAMPLE
   mysql> show binlog events;
   #+END_EXAMPLE

* 系统状态管理
** 查看所有已安装存储引擎
   
   #+BEGIN_EXAMPLE
   MySQL> show storage engines;
   #+END_EXAMPLE

   字段 Support 值如果为 DEFAULT 就表示这个似乎数据库默认存储引擎

** 查看服务器所支持的不同权限

   #+BEGIN_EXAMPLE
   MySQL> show privileges;
   #+END_EXAMPLE

** 查看所有服务器全局变量信息

   #+BEGIN_EXAMPLE
   MySQL> show global variables;
   #+END_EXAMPLE

** 查看某一个表的索引数据

   #+BEGIN_EXAMPLE
   MySQL> show index from tablename;
   #+END_EXAMPLE

** 查看所有存储过程状态

   #+BEGIN_EXAMPLE
   MySQL> show procedure status;
   #+END_EXAMPLE

** 查看所有会话连接信息
   
   查看当前系统中所有会话连接信息

   #+BEGIN_EXAMPLE
   MySQL> show full processlist;
   #+END_EXAMPLE

   Id 表示当前会话 ID 标识，User 表示连接登录的用户， Host 远程主机地址， db 当前连接的数据库， Time 会话连接时长， Info 正在执行的命令， Progress 表示 SQL 耗时时间。当发现某一个会话耗时太长，可以使用 kill 命令 + 会话ID，来强制终止会话。

   #+BEGIN_EXAMPLE
   MySQL> kill 12
   #+END_EXAMPLE

** 查看当前线程运行数

   使用下面这个命令可以查看当前 MySQL 服务器的线程数量

   #+BEGIN_EXAMPLE
   MySQL> show global status like '%threads_runn%';
   #+END_EXAMPLE

** 查询最后一个执行语句产生的错误信息

   #+BEGIN_EXAMPLE
   MySQL> show errors;
   #+END_EXAMPLE

** 查看最后一个执行的语句所产生的警告

   #+BEGIN_EXAMPLE
   MySQL> show warnings;
   #+END_EXAMPLE
** 检测数据库服务器是否存活

   #+BEGIN_EXAMPLE
   $ mysqladmin -h 192.168.1.100 -u root -p123456 ping
   #+END_EXAMPLE
** 查看 Master 日志文件列表

   #+BEGIN_EXAMPLE
   mysql> show master logs
   #+END_EXAMPLE
** 查看 mysql 事件信息

   #+BEGIN_EXAMPLE
   mysql> select * from mysql.event;
   #+END_EXAMPLE
