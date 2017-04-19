## PostgreSQL 9.6 编译安装

服务器环境为 CentOS 7.3 和 PostgreSQL 9.6.2。通常我们建议使用最新的 PostgreSQL 版本。首先在 PostgreSQL 官方网站下载最新版本的 PostgreSQL 源代码包。

### 安装 PostgreSQL 依赖开发包

```
$ gcc gcc-c++ make readline-devel zlib-devel openssl-devel perl-devel libxml2-devel
```

### 编译 PostgreSQL 源码

```
$ tar -xzvf postgresql-9.6.2.tar.gz
$ cd postgresql-9.6.2
$ ./configure --prefix=/usr/local/postgresql-9.6 --datadir=/var/pgsql/data --with-systemd
$ make
$ make install
```

### 创建运行用户和数据库目录

```
$ useradd -m -d /var/pgsql -c "postgresql databases user" postgres
$ mkdir -p /var/pgsql/data
$ chown -R postgres:postgres /var/pgsql/data
```

### 使用 initdb 初始化数据库

```
$ su - postgres
$ /usr/local/postgresql-9.6/bin/initdb --pgdata=/var/pgsql/data --encoding=UTF8
```

### 修改相关配置文件

pg_hba.conf

```
host    all             all             0.0.0.0/0            md5
```
postgresql.conf
```
listen_addresses = '*'
port = 5432
max_connections = 128
shared_buffers = 256MB
work_mem = 16MB
maintenance_work_mem = 64MB
effective_cache_size = 256MB
```

每个配置选项的值需要根据服务器的硬件配置而定，比如 CPU、内存、磁盘等等


### 启动 PostgreSQL 服务

```
$ /usr/local/postgresql-9.6/bin/pg_ctl -D /var/pgsql/pgdata -l /tmp/postgresql.log start
```

### systemd 服务脚本

```
[Unit]
Description=PostgreSQL database server
After=network.target

[Service]
Type=forking
User=postgres
Group=postgres
OOMScoreAdjust=-1000
TimeoutSec=300

ExecStart=/usr/bin/pg_ctl -D /var/pgsql/data -s -w -t 300 start
ExecStop=/usr/bin/pg_ctl -D /var/pgsql/data -s -m fast stop
ExecReload=/usr/bin/pg_ctl -D /var/pgsql/data -s reload

[Install]
WantedBy=multi-user.target
```
