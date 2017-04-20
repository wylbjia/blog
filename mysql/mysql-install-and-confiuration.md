# MySQL安装与配置
## MySQL 概述

MySQL 早年已经被 Oracle 公司收购，现有 MySQL 源代码已经没有以前那么开放，当年 MySQL 创始人从 Sun 公司离开之后，又重新开发了一个 MySQL 新的分支版本 MariaDB，名字取自作者女儿的名字。从 RedHat 7 和 CentOS 7 版本以后默认数据库改为了 MariaDB，MariaDB 目前拥有 MariaDB 5 和 MariaDB 10 两个版本，MariaDB 5 和 MySQL 5 除了名字不一样，使用方法基本没有区别。MariaDB 10 版本以上加入了一些新的特性和功能，这个版本开始和 MySQL 有些区别。使用 MariaDB 5 版本可以从 MySQL 无缝迁移到 MariaDB，

## 使用 yum 安装

在 RedHat、CentOS、Fedora 等发行版下，可以使用 yum 工具安装 MySQL

```
$ yum install -y mariadb mariadb-server mariadb-devel
$ chkconfig mysqld on
$ service mysqld start
```

## 源码编译安装
   
- CentOS 6.9
- MariaDB 5.5.54

### 安装 MariaDB 依赖包

```
$ yum install -y gcc gcc-c++ make bison bison-devel ncurses ncurses-devel zlib-devel
$ yum install -y cmake openssl openssl-devel jemalloc jemalloc-devel libevent libevent-devel

### 创建 MariaDB 运行用户

```
$ useradd -M -s /sbin/nologin -c "Mariadb Database User" mysql
```

### 下载 MariaDB 源码包，然后解压编译安装

```
$ wget https://mirrors.tuna.tsinghua.edu.cn/mariadb/mariadb-5.5.54/source/mariadb-5.5.54.tar.gz
$ tar -xzvf mariadb-5.5.54.tar.gz
$ cd mariadb-5.5.54
$ cmake . -DENABLE_DEBUG_SYNC:BOOL=OFF
          -DCMAKE_INSTALL_PREFIX:PATH=/usr/local/mysql
          -DMYSQL_DATADIR:PATH=/var/lib/mysql
          -DDEFAULT_CHARSET=utf8
          -DDEFAULT_COLLATION=utf8_general_ci
$ make
$ make install
```

### 创建 MariaDB 数据库存储目录

```
$ mkdir -p /var/lib/mysql
$ chown -R mysql:mysql /var/lib/mysql
```

### 开始初始化 MariaDB 数据库

```
$ /usr/local/mysql/scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/var/lib/mysql
```

### 添加 my.cnf 配置文件

```
$ cp /usr/local/mysql/support-files/my-large.cnf /etc/my.cnf
```

### 添加 MariaDB 服务启动脚本

```
$ cp /usr/local/mysql/support-files/mysql.server /etc/rc.d/init.d/mysqld
$ chkconfig --add mysqld
$ chkconfig mysqld on
$ service mysqld start
```

### 设置 MariaDB 数据库 root 密码

```
$ /usr/local/mysql/bin/mysqladmin -u root password '123456'
```
   
### 删除 MariaDB 中的无用账户

```
MySQL> delete from mysql.user where Password = '';
MySQL> flush privileges
```

### 允许 MariaDB 的 root 用户可以远程登录 (可选项)

```
MySQL> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
MySQL> flush privileges;
```

----------------------------------------------------------------------

Author: typefo <typefo@qq.com> 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)


