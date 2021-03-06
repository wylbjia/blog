# MySQL 高可用架构

MySQL 的高可用方案使用的是 Master 到 Master 双主相互同步复制，然后配合 Keepalived 做双机热备。并且只有其中一台对外提供服务，另外一台则为主备。两台服务器中的任何一台数据发生变化，都会同步到另外一台上。当任何一台发生故障，另外一台会自动抢占 VIP 地址，并自动成为主 Master 对外提供服务。

## MySQL双主复制

- 操作系统 CentOS 6.9 环境
- Master1 服务器 192.168.1.100
- Master2 服务器 192.168.1.200

**Master1 服务器配置**

在 Master1 上编辑 /etc/my.cnf 配置文件 

```
[mysqld]
server-id = 1                            # 服务器 ID 标识，同一局域网内不可与其他服务器重复
log-bin = mysql-bin                      # 开启 binlog 二进制日志，并指定日志文件名称
binlog_format = mixed                    # binlog 日志格式，mixed 表示混合模式
max_binlog_size = 500M                   # binlog 文件最大大小，超过大小会生成新文件
relay-log = slave-relay-bin              # 设置 relay 中继日志文件名
relay-log-index = slave-relay-bin.index  # 设置 relay 中继日志索引文件名
replicate_do_db = test                   # 指定哪些数据库需要同步复制
replicate_ignore_db = mysql              # 指定哪些数据库不需要同步
replicate_ignore_db = information_schema # 指定哪些数据库不需要同步
replicate_ignore_db = performance_schema # 指定哪些数据库不需要同步
log-slave-updates = 1                    # 如果这台 slave 同时也是其他服务器的 master 则必须开启此选项
slave-skip-errors = all                  # 忽略所有复制产生的错误 
```
  
在 Master1 上重启 mysql 数据库
  
```
$ service mysqld restart
```
  
在 Master1 上添加同步复制授权账号

```
mysql> grant replication slave on *.* to 'repluser'@'192.168.1.200' identified by '123456';
```

在 Master1 上重置并删除 binlog 日志

```
mysql> reset master;
```

在 Master2 查看 binlog 文件名和 pos 偏移点

```
mysql> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      245 |              |                  |
+------------------+----------+--------------+------------------+
```
 
在 Master1 上设置主从复制信息

```
mysql> change master to master_host="192.168.1.200",
                        master_port=3306,
                        master_user="repluser",
                        master_password="123456",
                        master_log_file="mysql-bin.000001",
                        master_log_pos=245;
```

在 Master1 上启动 Slave 同步复制

```
mysql> start slave;
```

在 Master1 上查看 Slave 同步复制状态

```
mysql> show slave status\G
```

**Master2 服务器配置**

在 Master2 上编辑 /etc/my.cnf 配置文件

```
[mysqld]
server-id = 2                            # 服务器 ID 标识，同一局域网内不可与其他服务器重复
log-bin = mysql-bin                      # 开启 binlog 二进制日志，并指定日志文件名称
binlog_format = mixed                    # binlog 日志格式，mixed 表示混合模式
max_binlog_size = 500M                   # binlog 文件最大大小，超过大小会生成新文件
relay-log = slave-relay-bin              # 设置 relay 中继日志文件名
relay-log-index = slave-relay-bin.index  # 设置 relay 中继日志索引文件名
replicate_do_db = test                   # 指定哪些数据库需要同步复制
replicate_ignore_db = mysql              # 指定哪些数据库不需要同步
replicate_ignore_db = information_schema # 指定哪些数据库不需要同步
replicate_ignore_db = performance_schema # 指定哪些数据库不需要同步
log-slave-updates = 1                    # 如果这台 slave 同时也是其他服务器的 master 则必须开启此选项
slave-skip-errors = all                  # 忽略所有复制产生的错误 
```
  
在 Master2 上重启 mysql 数据库
  
```
$ service mysqld restart
```
  
在 Master2 上添加同步复制授权账号

```
mysql> grant replication slave on *.* to 'repluser'@'192.168.1.100' identified by '123456';
```

在 Master2 上重置并删除 binlog 日志

```
mysql> reset master;
```

在 Master1 查看 binlog 文件名和 pos 偏移点

```
mysql> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      245 |              |                  |
+------------------+----------+--------------+------------------+
```
 
在 Master2 上设置主从复制信息

```
mysql> change master to master_host="192.168.1.100",
                        master_port=3306,
                        master_user="repluser",
                        master_password="123456",
                        master_log_file="mysql-bin.000001",
                        master_log_pos=245;
```

在 Master2 上启动 Slave 同步复制

```
mysql> start slave;
```

在 Master2 上查看 Slave 同步复制状态

```
mysql> show slave status\G
```

**MySQL 双主复制总结**

通常在 MySQL 双主同步复制的场景下，只让其中一台 Master 对外提供服务，另外一台则为 Master 备份。如果想让两台 Master 都对外提供读写服务，就需要重新设计和改变两台服务器的自增 ID 序列值。即便这样，还是无法完全避免一些其他方面的数据不一致问题。所以我们依旧不推荐在双主同步复制的情况下两台服务器都提供写操作的方案。

## MySQL 与 Keepalived
   
- VIP: 192.168.1.200
- Master1: 192.168.1.120
- Master2: 192.168.1.130

下载和安装 Keepalived 源码包
   
```
$ wget http://www.keepalived.org/software/keepalived-1.3.4.tar.gz
$ tar -xzvf keepalived-1.3.4.tar.gz
$ cd keepalived-1.3.4
$ ./configure
$ make
$ make install
```   

在 Master1 上修改 Keepalived 配置文件
   
```
global_defs {
    notification_email {
        acassen@firewall.loc
    }
   
    notification_email_from Alexandre.Cassen@firewall.loc
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id Master1
    vrrp_skip_check_adv_addr
    vrrp_garp_interval 0
    vrrp_gna_interval 0
}
   
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 60
    priority 100
    advert_int 1
    nopreempt
   
    authentication {
        auth_type PASS
        auth_pass 1111
    }
   
    virtual_ipaddress {
        192.168.1.200
    }
}
   
virtual_server 192.168.1.200 3306 {
    delay_loop 3
    persistence_timeout 50
    protocol TCP
   
    real_server 192.168.1.120 3306 {
        weight 1
        notify_down /etc/keepalived/check.sh
   
        TCP_CHECK {
            connect_timeout 10
            nb_get_retry 3
            delay_before_retry 3
            connect_port 3306
        }
    }
}
```

在 Master2 上修改 Keepalived 配置文件
   
```
global_defs {
    notification_email {
        acassen@firewall.loc
    }
   
    notification_email_from Alexandre.Cassen@firewall.loc
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id Master2
    vrrp_skip_check_adv_addr
    vrrp_garp_interval 0
    vrrp_gna_interval 0
}
   
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 60
    priority 85
    advert_int 1
   
    authentication {
        auth_type PASS
        auth_pass 1111
    }
   
    virtual_ipaddress {
        192.168.1.200
    }
}
   
virtual_server 192.168.1.200 3306 {
    delay_loop 3
    persistence_timeout 50
    protocol TCP
   
    real_server 192.168.1.130 3306 {
        weight 1
        notify_down /etc/keepalived/check.sh
   
        TCP_CHECK {
            connect_timeout 10
            nb_get_retry 3
            delay_before_retry 3
            connect_port 3306
        }
    }
}
```
   
服务检测脚本 /etc/keepalived/check.sh

```
#!/bin/sh
/usr/bin/pkill keepalived
```

给予服务器检测脚本可执行权限

```
$ chmod +x /etc/keepalived/check.sh
```

## binlog-do-db 注意事项

binlog-do-db 表示哪些数据库需要记录 binlog 日志。

- 当 binlog-format 为 statement 时

在使用了 use dbname 的情况下，如果 dbname 没有在 binlog-do-db 里，数据库的 DDL 和 DML 操作都不会被记录到 binlog 日志，即使使用了类似 test.table 的方式也不会被记录到 binlog 日志

- 当 binlog-format 为 row 时

在使用了 use dbname 的情况下，如果 dbname 没有在 binlog-do-db 里，数据库的 DDL 操作不会被记录到 binlog 日志。在使用或不使用 use dbname时， DML 操作都被记录到 binlog 日志

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
msyql> drop table table1;
```

在使用 `use db` 的情况下，并且 db 不在 replicate_do_db 里面，下列操作都不会被同步到 slave 从服务器

```
mysql> use db;
mysql> create table table1(id int, value varchar(8));
mysql> insert into table1 values(1, '1111111');
mysql> update table1 set value = 'aaaaaaa' where id = 1;
mysql> delete from table1 where id = 1;
msyql> drop table table1;
```

在没有使用 `use db` 的情况下，并且 db 在 replicate_do_db 里面，下列操作都不会被同步到 slave 从服务器

```
mysql> create table db.table1(id int, value varchar(8));
mysql> insert into db.table1 values(1, '1111111');
mysql> update db.table1 set value = 'aaaaaaa' where id = 1;
mysql> delete from db.table1 where id = 1;
msyql> drop table db.table1;
```

在没有使用 `use db` 的情况下，并且 db 不在 replicate_do_db 里面，下列操作都不会被同步到 slave 从服务器

```
mysql> create table db.table1(id int, value varchar(8));
mysql> insert into db.table1 values(1, '1111111');
mysql> update db.table1 set value = 'aaaaaaa' where id = 1;
mysql> delete from db.table1 where id = 1;
msyql> drop table db.table1;
```

-------------------------------------------------------------------------

By typefo typefo@qq.com Update: 2017-04-24 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)
