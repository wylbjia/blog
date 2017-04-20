#+title: MySQL备份与恢复
#+author: typefo
#+email: typefo@qq.com
#+options: ^:nil \n:t creator:nil
#+language: zh-CN

#+html_head: <link href="http://cdn.bootcss.com/bootstrap/2.3.2/css/bootstrap.min.css" rel="stylesheet">
#+html_head: <style type="text/css">body{padding:15px;margin:0 auto;width:1024px;font-size:15px;line-height:24px}</style>
#+html_head: <style type="text/css">h1{font-size:28px} h2{font-size:24px} h3{font-size:18px} h4{font-size:16px}</style>
#+html_head: <style type="text/css">p{text-indent:10px}</style>

* MySQL备份与恢复
** MySQL数据备份
   
   导出一个数据库
   
   #+BEGIN_EXAMPLE
   $ mysqldump -h 127.0.0.1 -u root -p --master-data=2 --single-transaction --add-drop-table --create-options --extended-insert
               --default-character-set=utf8
               --quick
               --databases
               dbname > database.sql
   #+END_EXAMPLE
   
   导出一个表
   
   #+BEGIN_EXAMPLE
   $ mysqldump -u root -p --opt --flush-logs root tablename > table.sql
   #+END_EXAMPLE
   
   
   将备份文件压缩
   
   #+BEGIN_EXAMPLE
   $ mysqldump -h 127.0.0.1 -u root -p123456 --databases dbname | gzip > database.sql.gz
   #+END_EXAMPLE

   对应的还原动作为

   #+BEGIN_EXAMPLE
   $ gunzip < database.sql.gz | mysql -u root -p123456 dbname
   #+END_EXAMPLE
   
** MySQL数据恢复

   导入数据库
   
   #+BEGIN_EXAMPLE
   MySQL> use 
   MySQL> source /root/database.sql
   #+END_EXAMPLE
   
   或者

   #+BEGIN_EXAMPLE
   $ mysql dbname < database.sql
   #+END_EXAMPLE
   
   直接从一个数据库向另一个数据库转储
   
   #+BEGIN_EXAMPLE
   $ mysqldump -u root -p --opt dbname | mysql --host 192.168.1.100 -C dbname
   #+END_EXAMPLE

** MySQL备份脚本
** 从Binlog恢复数据

   binlog 是 MySQL 的二进制日志文件，可以使用 mysqlbinlog 工具进行查看和恢复操作

   #+BEGIN_EXAMPLE
   $ mysqlbinlog [options] log-files
   #+END_EXAMPLE

   *恢复从起始 pos 点之后的数据* (文件中 at 点)

   #+BEGIN_EXAMPLE
   $ mysqlbinlog --start-position=4 /var/lib/mysql/mysql-bin.000001 > backup.sql
   #+END_EXAMPLE

   *恢复从起始 pos 点到结束 pos 之间的数据* (文件中 at 点)
   
   #+BEGIN_EXAMPLE
   $ mysqlbinlog --start-position=4 stop-position=35678 /var/lib/mysql/mysql-bin.000001 > backup.sql
   #+END_EXAMPLE
   
   *从数据库意外故障恢复*

   如果遇到服务器严重故障灾难，应该用最近一次使用 mysqldump 导出的完整备份恢复数据库

   #+BEGIN_EXAMPLE
   $ mysql -u root -p < backup.sql
   #+END_EXAMPLE
   
   然后使用从备份时间点之后的 binlog 日志文件把数据库恢复到最接近现在的可用状态，如有多个 binlog 文件，必须从最早生成的文件开始依次恢复

   #+BEGIN_EXAMPLE
   $ mysqlbinlog --start-position=4 /var/lib/mysql/mysql-bin.000002 | mysql -u root -p
   #+END_EXAMPLE

** mysqldump参数详解
   
   #+BEGIN_EXAMPLE
   -A, --all-databases
   #+END_EXAMPLE
   
   导出所有数据库
   
   #+BEGIN_EXAMPLE
   -Y, --all-tablespaces
   #+END_EXAMPLE
   
   导出全部表空间
   
   #+BEGIN_EXAMPLE
   -y, --no-tablespaces
   #+END_EXAMPLE
   
   不导出任何表空间信息
   
   #+BEGIN_EXAMPLE
   --add-drop-database
   #+END_EXAMPLE
   
   在创建数据库之前先用删除同名数据库
   
   #+BEGIN_EXAMPLE
   --add-drop-table
   #+END_EXAMPLE
   
   创建表之前先删除同名数据表，默认启用，使用 --skip-add-drop-table 取消
   
   #+BEGIN_EXAMPLE
   --add-locks
   #+END_EXAMPLE
   
   在导出表之前先将表加锁，之后再解锁，默认为开启，使用 --skip-add-locks 取消
   
   #+BEGIN_EXAMPLE
   --allow-keywords
   #+END_EXAMPLE
   
   允许创建与 mysql 内部关键词同名的列字段
   
   #+BEGIN_EXAMPLE
   --apply-slave-statements
   #+END_EXAMPLE
   
   在 'CHANGE MASTER' 前添加 'STOP SLAVE'，并且在导出的最后添加 'START SLAVE'
   
   #+BEGIN_EXAMPLE
   --character-sets-dir=name
   #+END_EXAMPLE
   
   字符集文件的目录
   
   #+BEGIN_EXAMPLE
   -i, --comments
   #+END_EXAMPLE
   
   在导出的 sql 文件中添加相关注释信息。默认为开启，可以用 --skip-comments 取消
   
   #+BEGIN_EXAMPLE
   --compatible=name
   #+END_EXAMPLE
   
   导出的数据将和其它数据库或旧版本的 MySQL 相兼容。其值可以为 ansi, mysql323, mysql40, postgresql, oracle, mssql, db2, maxdb, no_key_options,
   no_table_options, no_field_options 等，要使用多个值，用逗号将它们隔开。它并不保证能完全兼容，而是尽量兼容。
   
   #+BEGIN_EXAMPLE
   --compact
   #+END_EXAMPLE
   
   导出更少的输出信息(用于调试)。去掉注释和头尾等结构。可以使用选项：--skip-add-drop-table, --skip-add-locks, --skip-comments, --skip-disable-keys, --skip-set-charset
   
   #+BEGIN_EXAMPLE
   -c, --complete-insert
   #+END_EXAMPLE
   
   使用完整的 insert 语句(包含列名称)。这么做能提高插入效率，但是可能会受到 max_allowed_packet 参数的影响而导致插入失败。
   
   #+BEGIN_EXAMPLE
   -C, --compress
   #+END_EXAMPLE
   
   在客户端和服务器之间启用压缩传递所有信息
   
   #+BEGIN_EXAMPLE
   -a, --create-options
   #+END_EXAMPLE
   
   在 CREATE TABLE 语句中包括所有 MySQL 特性选项。默认开启，使用 --skip-create-options 取消
   
   #+BEGIN_EXAMPLE
   -B, --databases
   #+END_EXAMPLE
   
   导出几个数据库。参数后面所有名字参量都被看作数据库名。
   
   #+BEGIN_EXAMPLE
   --debug
   #+END_EXAMPLE
   
   输出debug信息，用于调试。默认值为 /tmp/mysqldump.trace
   
   #+BEGIN_EXAMPLE
   --debug-check
   #+END_EXAMPLE
   
   检查内存和打开文件使用说明并退出
   
   #+BEGIN_EXAMPLE
   --debug-info
   #+END_EXAMPLE
   
   输出调试信息并退出
   
   #+BEGIN_EXAMPLE
   --default-character-set=name
   #+END_EXAMPLE
   
   设置默认字符集，默认值为utf8
   
   #+BEGIN_EXAMPLE
   --delayed-insert
   #+END_EXAMPLE
   
   采用延时插入方式（INSERT DELAYED）导出数据
   
   #+BEGIN_EXAMPLE
   --delete-master-logs
   #+END_EXAMPLE
   
   master 备份后删除日志. 这个参数会自动激活 --master-data 选项
   
   #+BEGIN_EXAMPLE
   -K, --disable-keys
   #+END_EXAMPLE
   
   对于每个表，用 =/*!40000 ALTER TABLE tbl_name DISABLE KEYS */=  和  =/*!40000 ALTER TABLE tbl_name ENABLE KEYS */= 语句引用 INSERT 语句。这样可以更快地导入 dump 出来的文件，因为它是在插入所有行后创建索引的。该选项只适合 MyISAM 表，默认为打开状态。 默认开启, 使用 --skip-disable-keys 取消
   
   #+BEGIN_EXAMPLE
   --dump-slave
   #+END_EXAMPLE
   
   该选项将主的 binlog 位置和文件名追加到导出数据的文件中(show slave status). 设置为1时, 将会以 CHANGE MASTER命令输出到数据文件，设置为 2 时，会在 change 前加上注释。该选项将会打开 --lock-all-tables，除非 --single-transaction 被指定。该选项会自动关闭 --lock-tables 选项。默认值为 0
   
   #+BEGIN_EXAMPLE
   -E, --events
   #+END_EXAMPLE
   
   导出事件信息
   
   #+BEGIN_EXAMPLE
   -e, --extended-insert
   #+END_EXAMPLE
   
   使用具有多个 VALUES 列的 INSERT 语法。可使导出文件更小，加速导入速度。默认开启，使用 --skip-extended-insert 取消
   
   #+BEGIN_EXAMPLE
   --fields-terminated-by=name
   #+END_EXAMPLE
   
   导出文件中忽略给定字段。与 --tab 选项一起使用，不能用于 --databases 和 --all-databases 选项
   
   #+BEGIN_EXAMPLE
   --fields-enclosed-by=name
   #+END_EXAMPLE
   
   输出文件中的各个字段用给定字符包裹。与--tab选项一起使用，不能用于--databases和--all-databases选项
   
   #+BEGIN_EXAMPLE
   --fields-optionally-enclosed-by=name
   #+END_EXAMPLE
   
   输出文件中的各个字段用给定字符选择性包裹。与--tab选项一起使用，不能用于--databases和--all-databases选项
   
   #+BEGIN_EXAMPLE
   --fields-escaped-by=name*
   #+END_EXAMPLE
   
   输出文件中的各个字段忽略给定字符。与--tab选项一起使用，不能用于--databases和--all-databases选项
   
   #+BEGIN_EXAMPLE
   -F,--flush-logs
   #+END_EXAMPLE
   开始导出之前刷新日志。请注意：假如一次导出多个数据库(使用选项--databases或者--all-databases)，将会逐个数据库刷新日志。除使用--lock-all-tables或者--master-data外。在这种情况下，日志将会被刷新一次，相应的所以表同时被锁定。因此，如果打算同时导出和刷新日志应该使用--lock-all-tables 或者--master-data 和--flush-logs。
   
   #+BEGIN_EXAMPLE
   --flush-privileges
   #+END_EXAMPLE
   
   在导出mysql数据库之后，发出一条FLUSH  PRIVILEGES 语句。为了正确恢复，该选项应该用于导出mysql数据库和依赖mysql数据库数据的任何时候。
   
   #+BEGIN_EXAMPLE
   -f, --force
   #+END_EXAMPLE
   
   在导出过程中忽略出现的SQL错误。
   
   #+BEGIN_EXAMPLE
   -h, --host=name
   #+END_EXAMPLE
   
   数据库主机地址
   
   #+BEGIN_EXAMPLE
   --ignore-table=name
   #+END_EXAMPLE
   
   不导出指定表。指定忽略多个表时，需要重复多次，每次一个表。每个表必须同时指定数据库和表名。例如： --ignore-table=database.table
   
   #+BEGIN_EXAMPLE
   --include-master-host-port
   #+END_EXAMPLE
   
   在--dump-slave产生的'CHANGE  MASTER TO..'语句中增加'MASTER_HOST=<host>，MASTER_PORT=<port>'  
   
   #+BEGIN_EXAMPLE
   --insert-ignore
   #+END_EXAMPLE
   
   在插入行时使用 INSERT IGNORE 语句
   
   #+BEGIN_EXAMPLE
   --lines-terminated-by=name
   #+END_EXAMPLE
   
   输出文件的每行用给定字符串划分。与--tab选项一起使用，不能用于--databases和--all-databases选项。
   
   #+BEGIN_EXAMPLE
   -x, --lock-all-tables
   #+END_EXAMPLE
   
   提交请求锁定所有数据库中的所有表，以保证数据的一致性。这是一个全局读锁，并且自动关闭--single-transaction 和--lock-tables 选项。
   
   #+BEGIN_EXAMPLE
   -l, --lock-tables
   #+END_EXAMPLE
   
   开始导出前，锁定所有表。用READ  LOCAL锁定表以允许MyISAM表并行插入。对于支持事务的表例如InnoDB和BDB，--single-transaction是一个更好的选择，因为它根本不需要锁定表。
   请注意当导出多个数据库时，--lock-tables分别为每个数据库锁定表。因此，该选项不能保证导出文件中的表在数据库之间的逻辑一致性。不同数据库表的导出状态可以完全不同。默认开启，使用  use --skip-lock-tables 取消
   
   #+BEGIN_EXAMPLE
   --log-error=name
   #+END_EXAMPLE
   
   附加警告和错误信息到给定文件
   
   #+BEGIN_EXAMPLE
   --master-data
   #+END_EXAMPLE
   
   mysqldump 导出数据时，当这个参数的值为 1 的时候，mysqldump 出来的文件就会包括 CHANGE MASTER TO 这个语句，CHANGE MASTER TO 后面紧接着就是 file 和 position 的记录，在 slave 上导入数据时就会执行这个语句，salve 就会根据指定这个文件位置从 master 端复制 binlog。默认情况下这个值是 1 当这个值是2的时候，chang master to 也是会写到dump文件里面去的，但是这个语句是被注释的状态。
   
   #+BEGIN_EXAMPLE
   --max-allowed-packet
   #+END_EXAMPLE
   
   服务器发送和接受的最大包长度
   
   #+BEGIN_EXAMPLE
   --net-buffer-length
   #+END_EXAMPLE
   
   TCP/IP和socket连接的缓存大小。
   
   #+BEGIN_EXAMPLE
   --no-autocommit
   #+END_EXAMPLE
   
   使用autocommit/commit 语句包裹表。
   
   #+BEGIN_EXAMPLE
   -n, --no-create-db
   #+END_EXAMPLE
   
   只导出数据，而不添加CREATE DATABASE 语句。
   
   #+BEGIN_EXAMPLE
   -t, --no-create-info
   #+END_EXAMPLE
   
   只导出数据，而不添加CREATE TABLE 语句。
   
   #+BEGIN_EXAMPLE
   -d, --no-data
   #+END_EXAMPLE
   
   不导出任何数据，只导出数据库表结构。
   
   #+BEGIN_EXAMPLE
   -N, --no-set-names
   #+END_EXAMPLE
   
   等同于--skip-set-charset
   
   #+BEGIN_EXAMPLE
   --opt
   #+END_EXAMPLE
   
   等同于 --add-drop-table, --add-locks, --create-options, --quick, --extended-insert, --lock-tables,  --set-charset, --disable-keys 该选项默认开启, 可以用--skip-opt禁用.
   
   #+BEGIN_EXAMPLE
   --order-by-primary
   #+END_EXAMPLE
   
   如果存在主键，或者第一个唯一键，对每个表的记录进行排序。在导出MyISAM表到InnoDB表时有效，但会使得导出工作花费很长时间。 
   
   #+BEGIN_EXAMPLE
   -p, --password[=name]
   #+END_EXAMPLE
   
   数据库账户密码
   
   #+BEGIN_EXAMPLE
   -P, --port
   #+END_EXAMPLE
   
   数据库服务器端口
   
   #+BEGIN_EXAMPLE
   --protocol=name
   #+END_EXAMPLE
   
   使用的连接协议，包括：tcp, socket, pipe, memory
   
   #+BEGIN_EXAMPLE
   -q, --quick
   #+END_EXAMPLE
   
   不缓冲查询，直接导出到标准输出。默认为打开状态，使用--skip-quick取消该选项
   
   #+BEGIN_EXAMPLE
   -Q, --quote-names
   #+END_EXAMPLE
   
   使用（`）引起表和列名。默认为打开状态，使用--skip-quote-names取消该选项。
   
   #+BEGIN_EXAMPLE
   --replace
   #+END_EXAMPLE
   
   使用REPLACE INTO 取代INSERT INTO
   
   #+BEGIN_EXAMPLE
   -r, --result-file=name
   #+END_EXAMPLE
   
   直接输出到指定文件中。该选项应该用在使用回车换行对（\\r\\n）换行的系统上（例如：DOS，Windows）。该选项确保只有一行被使用。
   
   #+BEGIN_EXAMPLE
   -R, --routines
   #+END_EXAMPLE
   
   导出存储过程以及自定义函数
   
   #+BEGIN_EXAMPLE
   --set-charset
   #+END_EXAMPLE
   
   添加'SET NAMES  default_character_set'到输出文件。默认为打开状态，使用--skip-set-charset关闭选项。
   
   #+BEGIN_EXAMPLE
   --single-transaction
   #+END_EXAMPLE
   
   该选项在导出数据之前提交一个BEGIN SQL语句，BEGIN 不会阻塞任何应用程序且能保证导出时数据库的一致性状态。它只适用于多版本存储引擎，仅InnoDB。本选项和--lock-tables 选项是互斥的，因为LOCK  TABLES 会使任何挂起的事务隐含提交。要想导出大表的话，应结合使用--quick 选项。
   
   #+BEGIN_EXAMPLE
   --dump-date
   #+END_EXAMPLE
   
   将导出时间添加到输出文件中。默认为打开状态，使用--skip-dump-date关闭选项
   
   #+BEGIN_EXAMPLE
   --skip-opt
   #+END_EXAMPLE
   
   禁用–opt选项
   
   #+BEGIN_EXAMPLE
   -S, --socket=name
   #+END_EXAMPLE
   
   指定连接mysql的socket文件位置，默认路径/tmp/mysql.sock
   
   #+BEGIN_EXAMPLE
   -T, --tab=name
   #+END_EXAMPLE
   
   为每个表在给定路径创建tab分割的文本文件。注意：仅仅用于mysqldump和mysqld服务器运行在相同机器上。注意使用--tab不能指定--databases参数
   
   #+BEGIN_EXAMPLE
   --tables
   #+END_EXAMPLE
   
   覆盖--databases (-B)参数，指定需要导出的表名，在后面的版本会使用table取代tables。
   
   #+BEGIN_EXAMPLE
   --triggers
   #+END_EXAMPLE
   
   导出触发器。该选项默认启用，用--skip-triggers禁用它。
   
   #+BEGIN_EXAMPLE
   --tz-utc
   #+END_EXAMPLE
   
   在导出顶部设置时区TIME_ZONE='+00:00' ，以保证在不同时区导出的TIMESTAMP 数据或者数据被移动其他时区时的正确性。
   
   #+BEGIN_EXAMPLE
   -u, --user=name
   #+END_EXAMPLE
   
   指定连接的用户名
   
   #+BEGIN_EXAMPLE
   --verbose, --v
   #+END_EXAMPLE
   
   输出多种平台信息。
   
   #+BEGIN_EXAMPLE
   --version, -V
   #+END_EXAMPLE
   
   输出mysqldump版本信息并退出
   
   #+BEGIN_EXAMPLE
   --where, -w
   #+END_EXAMPLE
   
   只转储给定的WHERE条件选择的记录。请注意如果条件包含命令解释符专用空格或字符，一定要将条件引用起来。
   
   #+BEGIN_EXAMPLE
   --xml, -X
   #+END_EXAMPLE
   
   导出XML格式.
   
   #+BEGIN_EXAMPLE
   --plugin_dir
   #+END_EXAMPLE
   
   客户端插件的目录，用于兼容不同的插件版本。
   
   #+BEGIN_EXAMPLE
   --default_auth
   #+END_EXAMPLE
   
   客户端插件默认使用权限。
   
