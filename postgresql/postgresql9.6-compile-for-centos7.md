## PostgreSQL 9.6 编译安装 CentOS 7

服务器环境为 CentOS 7.3 和 PostgreSQL 9.6.2。通常我们建议使用最新的 PostgreSQL 版本。首先在 PostgreSQL 官方网站下载最新版本的 PostgreSQL 源代码包。

#### 安装 PostgreSQL 依赖开发包

```
$ yum install -y gcc gcc-c++ make readline-devel zlib-devel openssl-devel perl-devel libxml2-devel systemd-devel
```

#### 编译 PostgreSQL 源码

```
$ tar -xzvf postgresql-9.6.2.tar.gz
$ cd postgresql-9.6.2
$ ./configure --prefix=/usr/local/postgresql-9.6 --with-systemd
$ make
$ make install
```

#### 创建运行用户和数据库目录

```
$ useradd -m -d /var/pgsql -c "postgresql databases user" postgres
$ mkdir -p /var/pgsql/data
$ chown -R postgres:postgres /var/pgsql/data
```

#### 使用 initdb 初始化数据库

```
$ su - postgres
$ /usr/local/postgresql-9.6/bin/initdb --pgdata=/var/pgsql/data --encoding=UTF8
```

#### 修改相关配置文件

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


#### 启动 PostgreSQL 服务

```
$ /usr/local/postgresql-9.6/bin/pg_ctl -D /var/pgsql/pgdata -l /tmp/postgresql.log start
```

#### systemd 服务脚本

```
[Unit]
Description=PostgreSQL database server
After=network.target

[Service]
Type=forking
User=postgres
Group=postgres
LimitNPROC=65535
LimitNOFILE=102400
OOMScoreAdjust=-1000
TimeoutSec=300

ExecStart=/usr/local/postgresql-9.6/bin/pg_ctl -D /var/pgsql/data -s -w -t 300 start
ExecStop=/usr/local/postgresql-9.6/bin/pg_ctl -D /var/pgsql/data -s -m fast stop
ExecReload=/usr/local/postgresql-9.6/bin/pg_ctl -D /var/pgsql/data -s reload

[Install]
WantedBy=multi-user.target
```

#### 内核参数优化

sysctl.conf

```
fs.nr_open = 2048000
fs.file-max = 1024000
fs.aio-max-nr = 1048576
net.ipv4.tcp_fin_timeout = 5
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_tw_reuse = 1
```

limits.conf

```
* soft    nofile  1024000
* hard    nofile  1024000
* soft    nproc   unlimited
* hard    nproc   unlimited
* soft    core    unlimited
* hard    core    unlimited
* soft    memlock unlimited
* hard    memlock unlimited
```

-----------------------------------

Author: typefo <typefo@qq.com> 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)

