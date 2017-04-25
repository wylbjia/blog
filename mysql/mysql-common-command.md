# MySQL 常用命令

本文档列出了一些日常使用到的 mysql 常用命令

## 账户管理

- 查询所有用户账户

```
MySQL> select Host,User,Password from mysql.user;
```

- 创建一个用户账户

```
MySQL> CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

`username` 表示账户名称， `host` 表示主机地址，百分号 `%` 表示任何主机， `password` 表示用户密码

- 创建一个 admin 用户账户，密码为 '123456'，并且可以从任何主机登录

```
MySQL> CREATE USER 'admin'@'%' IDENTIFIED BY '123456';
```
   
- 用户账户授权

GRANT 用户授权命令的基本格式

```
MySQL> GRANT privileges ON databasename.tablename TO 'username'@'host'
```

`privileges` 表示授权的操作，比如 SELECT, INSERT, UPDATE, DELETE 等等, ALL 代表所有操作
`databasename` 表示所需要授权的数据库名称，可用星号 * 表示所有数据库
`tablename` 表示所需要授权的表名称，可用星号 * 表示所有表
`username` 授权的用户名
`host` 授权的用户主机地址，百分号 '%' 表示可从任何主机连接登录

注意: 用以上命令授权的用户不能给其它用户授权,如果想让该用户可以授权,用以下命令:

```
MySQL> GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;
```

- 将数据库 test 的 select 权限赋予给 admin 用户

```
MySQL> GRANT SELECT ON test.* TO 'admin'@'192.168.1.130'
```

- 授予所有权限给 admin 用户

```
MySQL> GRANT ALL ON *.* TO 'admin'@'192.168.1.130'
```

- 修改一个用户密码
  
```
MySQL> SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
```

`username` 需要修改的用户帐户， `host` 对应用户的允许连接登录主机， `newpassword` 新的密码

- 修改当前登录用户自己的密码

```
MySQL> SET PASSWORD = PASSWORD("newpassword");
```
  
`newpasword` 表示新的密码

- 撤销一个用户权限

```
MySQL> REVOKE privilege ON databasename.tablename FROM 'username'@'host';
```

`privileges` 授权操作，比如 SELECT, INSERT, UPDATE, DELETE 等等, ALL 代表所有操作

`databasename` 数据库名称，可用星号 * 表示所有数据库

`tablename` 表名称，可用星号 * 表示所有表

`username` 用户账户

`host` 户主机地址，百分号 '%' 任何主机

- 撤销用户 admin 对数据库 test 的 SELECT 权限

```
MySQL> REVOKE SELECT ON test.* FROM 'admin'@'192.168.1.130';
```

- 删除一个用户账户的基本命令格式

```
MySQL> DROP USER 'username'@'host';
```

比如删除 admin 用户账户

```
MySQL> DROP USER 'admin'@'192.168.1.130';
```

- 刷新权限表，所有对用户的操作，最后都需要使用下面的命令刷新数据库权限表才能生效

```
MySQL> flush privileges;
```

- 查看用户权限

```
MySQL> show grants for root;
```

## 数据库管理

- 查看某个数据库是如何创建的

```
MySQL> show create database databasename;
```

- 将 SQL 文件导入数据库

```
mysql> use database;
mysql> source /root/backup.sql;
```

- 查看某个表是如何创建的

```
MySQL> show create table tablename;
```

- 查看一个表的列字段名称

```
MySQL> show columns from tablename;
```

**mysql 锁表和解锁**

- 当需要对数据库做备份或者同步的时候，需要事先将表加锁，防止其他客户端写入数据，导致数据不一致

```
mysql> flush table with read lock;
```

- 将一个表解锁

```
mysql> unlock table;
```

- 清空表数据但不删除表

```
mysql> delete from tablename;
```

## 存储过程

- 查看所有存储过程

```
mysql> show procedure status;
```

或者

```
mysql> select * from mysql.proc;
```

## 触发器管理

- 查看所有触发器

```
mysql> select * from information_schema.triggers;
```

## 函数管理

- 查看所有函数

```
mysql> show function status;
```

## 日志管理

**Binlog日志管理**
   
- 删除所有 binlog 日志

```
mysql> reset master;
```

- 删除指定 binlog 日志之前的文件

```
mysql> purge master logs to 'mysql-bin.000005';
```

上面的命令将会删除 mysql-bin.000005 之前生成的 binlog 日志文件，mysql-bin.000005 本身不会被删除

- 删除指定时间之前生成的 binlog 日志文件

```
mysql> purge master logs before '2017-03-15 18:00:00';
```

上面的命令将会删除 '2017-03-15 18:00:00' 时间之前生成的 binlog 日志文件，这个时间点生成文件不会被删除

## 主从同步管理

- 刷新 binlog 日志。此操作会使服务器重新建立新的 binlog 日志文件

```
mysql> flush logs
```

- 重置 Master，此操作会清空所有 bin-log 日志，只保留 mysql-bin.000001 文件

```
mysql> reset master
```

- 查看 binlog 记录事件

```
mysql> show binlog events;
```

## 系统状态管理

- 查看所有已安装存储引擎，字段 Support 值如果为 DEFAULT 就表示这个似乎数据库默认存储引擎
   
```
MySQL> show storage engines;
```
   
- 查看服务器所支持的不同权限

```
MySQL> show privileges;
```

- 查看所有服务器全局变量信息

```
MySQL> show global variables;
```

- 查看某一个表的索引数据

```
MySQL> show index from tablename;
```

- 查看所有存储过程状态

```
MySQL> show procedure status;
```

- 查看当前系统中所有会话连接信息

```
MySQL> show full processlist;
```

`Id` 表示当前会话 ID 标识，`User` 表示连接登录的用户， `Host` 远程主机地址， `db` 当前连接的数据库， `Time` 会话连接时长， `Info` 正在执行的命令， `Progress` 表示 SQL 耗时时间。当发现某一个会话耗时太长，可以使用 kill 命令 + 会话ID，来强制终止会话。

```
MySQL> kill 12
```

- 查看当前 MySQL 服务器的线程数量

```
MySQL> show global status like '%threads_runn%';
```

- 查询最后一个执行语句产生的错误信息

```
MySQL> show errors;
```

- 查看最后一个执行的语句所产生的警告

```
MySQL> show warnings;
```

- 检测数据库服务器是否存活

```
$ mysqladmin -h 192.168.1.100 -u root -p123456 ping
```

- 查看 Master 日志文件列表

```
mysql> show master logs
```

- 查看 mysql 事件信息

```
mysql> select * from mysql.event;
```

-------------------------------------------------------------------------------

By typefo typefo@qq.com Update: 2017-04-25 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)

