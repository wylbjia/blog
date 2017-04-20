### MySQL 数据库备份与恢复

有时候我们需要经常对 MySQL 的数据进行备份。以便在灾难发生时能恢复数据，保证线上业务的正常运行。通常我们使用 mysqldump 工具来进行 MySQL 的数据备份操作

#### MySQL数据备份
   
**导出一个数据库**

```
$ mysqldump -h 127.0.0.1 -u root -p --master-data=2 --single-transaction --add-drop-table --create-options --extended-insert
            --default-character-set=utf8
            --quick
            --databases
            dbname > database.sql
```
   
**导出一个表**
   
```
$ mysqldump -u root -p --opt --flush-logs root tablename > table.sql
```
   
**将备份文件压缩**
   
```
$ mysqldump -h 127.0.0.1 -u root -p123456 --databases dbname | gzip > database.sql.gz
```

**对应的还原动作为**

```
$ gunzip < database.sql.gz | mysql -u root -p123456 dbname
```

#### MySQL数据恢复

**导入数据库**

```
MySQL> use 
MySQL> source /root/database.sql
```
   
或者

```
$ mysql dbname < database.sql
```
   
**直接从一个数据库向另一个数据库转储**

```
$ mysqldump -u root -p --opt dbname | mysql --host 192.168.1.100 -C dbname
```

#### 从 binlog 恢复数据

binlog 是 MySQL 的二进制日志文件，可以使用 mysqlbinlog 工具进行查看和恢复操作

```
$ mysqlbinlog [options] log-files
```

**恢复从起始 pos 点之后的数据** (文件中 at 点)

```
$ mysqlbinlog --start-position=4 /var/lib/mysql/mysql-bin.000001 > backup.sql
```

**恢复从起始 pos 点到结束 pos 之间的数据** (文件中 at 点)
   
```
$ mysqlbinlog --start-position=4 stop-position=35678 /var/lib/mysql/mysql-bin.000001 > backup.sql
```
   
**从数据库意外故障恢复**

如果遇到服务器严重故障灾难，应该用最近一次使用 mysqldump 导出的完整备份恢复数据库

```
$ mysql -u root -p < backup.sql
```
   
然后使用从备份时间点之后的 binlog 日志文件把数据库恢复到最接近现在的可用状态，如有多个 binlog 文件，必须从最早生成的文件开始依次恢复

```
   $ mysqlbinlog --start-position=4 /var/lib/mysql/mysql-bin.000002 | mysql -u root -p
```

#### mysqldump 参数详解
   
```
-A, --all-databases
```
   
导出所有数据库
   
```
-Y, --all-tablespaces
```
   
导出全部表空间
   
```
   -y, --no-tablespaces
```

不导出任何表空间信息

```
--add-drop-database
```
   
在创建数据库之前先用删除同名数据库
   
```
--add-drop-table
```
   
创建表之前先删除同名数据表，默认启用，使用 --skip-add-drop-table 取消
   
```
--add-locks
```
   
在导出表之前先将表加锁，之后再解锁，默认为开启，使用 --skip-add-locks 取消
   
```
--allow-keywords
```
   
允许创建与 mysql 内部关键词同名的列字段
   
```
--apply-slave-statements
```
   
在 'CHANGE MASTER' 前添加 'STOP SLAVE'，并且在导出的最后添加 'START SLAVE'
   
```
--character-sets-dir=name
```
   
字符集文件的目录
   
```
-i, --comments
```
   
在导出的 sql 文件中添加相关注释信息。默认为开启，可以用 --skip-comments 取消
   
```
--compatible=name
```
   
导出的数据将和其它数据库或旧版本的 MySQL 相兼容。其值可以为 ansi, mysql323, mysql40, postgresql, oracle, mssql, db2, maxdb, no_key_options, no_table_options,no_field_options 等，要使用多个值，用逗号将它们隔开。它并不保证能完全兼容，而是尽量兼容。
   
```
--compact
```
   
导出更少的输出信息(用于调试)。去掉注释和头尾等结构。可以使用选项：--skip-add-drop-table, --skip-add-locks, --skip-comments, --skip-disable-keys, --skip-set-charset
   
```
-c, --complete-insert
```
   
使用完整的 insert 语句(包含列名称)。这么做能提高插入效率，但是可能会受到 max_allowed_packet 参数的影响而导致插入失败。
   
```
-C, --compress
```
   
在客户端和服务器之间启用压缩传递所有信息
   
```
-a, --create-options
```
   
在 CREATE TABLE 语句中包括所有 MySQL 特性选项。默认开启，使用 --skip-create-options 取消
   
```
-B, --databases
```
   
导出几个数据库。参数后面所有名字参量都被看作数据库名。
   
```
--debug
```
   
输出debug信息，用于调试。默认值为 /tmp/mysqldump.trace
   
```
--debug-check
```
   
检查内存和打开文件使用说明并退出
   
```
--debug-info
```
   
输出调试信息并退出
   
```
--default-character-set=name
```
   
设置默认字符集，默认值为utf8
   
```
--delayed-insert
```
   
采用延时插入方式（INSERT DELAYED）导出数据
   
```
--delete-master-logs
```

master 备份后删除日志. 这个参数会自动激活 --master-data 选项
   
```
-K, --disable-keys
```
对于每个表，用 =/*!40000 ALTER TABLE tbl_name DISABLE KEYS */=  和  =/*!40000 ALTER TABLE tbl_name ENABLE KEYS */= 语句引用 INSERT 语句。这样可以更快地导入 dump出来的文件，因为它是在插入所有行后创建索引的。该选项只适合 MyISAM 表，默认为打开状态。 默认开启, 使用 --skip-disable-keys 取消
   
```
--dump-slave
```
   
该选项将主的 binlog 位置和文件名追加到导出数据的文件中(show slave status). 设置为1时, 将会以 CHANGE MASTER命令输出到数据文件，设置为 2 时，会在 change 前加上注释。该选项将会打开--lock-all-tables，除非 --single-transaction 被指定。该选项会自动关闭 --lock-tables 选项。默认值为 0

```
-E, --events
```
   
导出事件信息
   
```
-e, --extended-insert
```
   
使用具有多个 VALUES 列的 INSERT 语法。可使导出文件更小，加速导入速度。默认开启，使用 --skip-extended-insert 取消
   
```
--fields-terminated-by=name
```
   
导出文件中忽略给定字段。与 --tab 选项一起使用，不能用于 --databases 和 --all-databases 选项
   
```
--fields-enclosed-by=name
```
   
输出文件中的各个字段用给定字符包裹。与--tab选项一起使用，不能用于--databases和--all-databases选项
   
```
--fields-optionally-enclosed-by=name
```
   
输出文件中的各个字段用给定字符选择性包裹。与--tab选项一起使用，不能用于--databases和--all-databases选项
   
```
--fields-escaped-by=name*
```
   
输出文件中的各个字段忽略给定字符。与--tab选项一起使用，不能用于--databases和--all-databases选项
   
```
-F,--flush-logs
```
 开始导出之前刷新日志。请注意：假如一次导出多个数据库(使用选项--databases或者--all-databases)，将会逐个数据库刷新日志。除使用--lock-all-tables或者--master-data外。在这种情况下，日志将会被刷新一次，相应的所以表同时被锁定。因此，如果打算同时导出和刷新日志应该使用--lock-all-tables 或者--master-data 和--flush-logs。
   
```
--flush-privileges
```
   
在导出mysql数据库之后，发出一条FLUSH  PRIVILEGES 语句。为了正确恢复，该选项应该用于导出mysql数据库和依赖mysql数据库数据的任何时候。
   
```
-f, --force
```
   
在导出过程中忽略出现的SQL错误。
   
```
-h, --host=name
```
   
数据库主机地址
   
```
--ignore-table=name
```
   
不导出指定表。指定忽略多个表时，需要重复多次，每次一个表。每个表必须同时指定数据库和表名。例如： --ignore-table=database.table
   
```
--include-master-host-port
```
   
在--dump-slave产生的'CHANGE  MASTER TO..'语句中增加'MASTER_HOST=<host>，MASTER_PORT=<port>'  
   
```
--insert-ignore
```
   
在插入行时使用 INSERT IGNORE 语句
   
```
--lines-terminated-by=name
```
   
输出文件的每行用给定字符串划分。与--tab选项一起使用，不能用于--databases和--all-databases选项。
   
```
-x, --lock-all-tables
```
   
 提交请求锁定所有数据库中的所有表，以保证数据的一致性。这是一个全局读锁，并且自动关闭--single-transaction 和--lock-tables 选项。
   
```
-l, --lock-tables
```
   
开始导出前，锁定所有表。用READ  LOCAL锁定表以允许MyISAM表并行插入。对于支持事务的表例如InnoDB和BDB，--single-transaction是一个更好的选择，因为它根本不需要锁定表。请注意当导出多个数据库时，--lock-tables分别为每个数据库锁定表。因此，该选项不能保证导出文件中的表在数据库之间的逻辑一致性。不同数据库表的导出状态可以完全不同。默认开启，使用  use --skip-lock-tables 取消
   
```
--log-error=name
```
   
附加警告和错误信息到给定文件
   
```
--master-data
```
   
mysqldump 导出数据时，当这个参数的值为 1 的时候，mysqldump 出来的文件就会包括 CHANGE MASTER TO 这个语句，CHANGE MASTER TO 后面紧接着就是 file 和 position 的记录，在 slave上导入数据时就会执行这个语句，salve 就会根据指定这个文件位置从 master 端复制 binlog。默认情况下这个值是 1 当这个值是2的时候，chang master to也是会写到dump文件里面去的，但是这个语句是被注释的状态。
   
```
--max-allowed-packet
```
   
服务器发送和接受的最大包长度
   
```
--net-buffer-length
```
   
TCP/IP和socket连接的缓存大小。
   
```
--no-autocommit
```
   
使用autocommit/commit 语句包裹表。
   
```
-n, --no-create-db
```
   
只导出数据，而不添加CREATE DATABASE 语句。
   
```
-t, --no-create-info
```
   
只导出数据，而不添加CREATE TABLE 语句。
   
```
-d, --no-data
```
   
不导出任何数据，只导出数据库表结构。
   
```
-N, --no-set-names
```
   
等同于 --skip-set-charset
   
```
--opt
```
等同于 --add-drop-table, --add-locks, --create-options, --quick, --extended-insert, --lock-tables,  --set-charset, --disable-keys 该选项默认开启, 可以用--skip-opt禁用.
   
```
--order-by-primary
```
   
如果存在主键，或者第一个唯一键，对每个表的记录进行排序。在导出MyISAM表到InnoDB表时有效，但会使得导出工作花费很长时间。 
   
```
-p, --password[=name]
```
   
数据库账户密码
   
```
-P, --port
```
   
数据库服务器端口
   
```
--protocol=name
```
   
使用的连接协议，包括：tcp, socket, pipe, memory
   
```
-q, --quick
```
   
不缓冲查询，直接导出到标准输出。默认为打开状态，使用--skip-quick取消该选项
   
```
-Q, --quote-names
```
   
使用（`）引起表和列名。默认为打开状态，使用--skip-quote-names取消该选项。
   
```
--replace
```
   
使用REPLACE INTO 取代INSERT INTO
   
```
-r, --result-file=name
```
   
直接输出到指定文件中。该选项应该用在使用回车换行对（\\r\\n）换行的系统上（例如：DOS，Windows）。该选项确保只有一行被使用。
   
```
-R, --routines
```
   
导出存储过程以及自定义函数
   
```
--set-charset
```
   
添加'SET NAMES  default_character_set'到输出文件。默认为打开状态，使用--skip-set-charset关闭选项。
   
```
--single-transaction
```
   
该选项在导出数据之前提交一个BEGIN SQL语句，BEGIN 不会阻塞任何应用程序且能保证导出时数据库的一致性状态。它只适用于多版本存储引擎，仅InnoDB。本选项和--lock-tables选项是互斥的，因为LOCK  TABLES会使任何挂起的事务隐含提交。要想导出大表的话，应结合使用--quick 选项。
   
```
--dump-date
```
   
将导出时间添加到输出文件中。默认为打开状态，使用--skip-dump-date关闭选项
   
```
--skip-opt
```
   
禁用–opt选项
   
```
-S, --socket=name
```
   
指定连接mysql的socket文件位置，默认路径/tmp/mysql.sock
   
```
-T, --tab=name
```
   
为每个表在给定路径创建tab分割的文本文件。注意：仅仅用于mysqldump和mysqld服务器运行在相同机器上。注意使用--tab不能指定--databases参数
   
```
--tables
```
   
覆盖--databases (-B)参数，指定需要导出的表名，在后面的版本会使用table取代tables。
   
```
--triggers
```
   
导出触发器。该选项默认启用，用--skip-triggers禁用它。
   
```
--tz-utc
```
   
在导出顶部设置时区TIME_ZONE='+00:00' ，以保证在不同时区导出的TIMESTAMP 数据或者数据被移动其他时区时的正确性。
   
```
-u, --user=name
```
   
指定连接的用户名
   
```
--verbose, --v
```
   
输出多种平台信息。
   
```
--version, -V
```
   
输出mysqldump版本信息并退出
   
```
--where, -w
```
   
只转储给定的WHERE条件选择的记录。请注意如果条件包含命令解释符专用空格或字符，一定要将条件引用起来。
   
```
--xml, -X
```
   
导出XML格式.
   
```
--plugin_dir
```
   
客户端插件的目录，用于兼容不同的插件版本。
   
```
--default_auth
```
   
客户端插件默认使用权限。

---------------------------------------------------------

Author: typefo <typefo@qq.com> 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)

