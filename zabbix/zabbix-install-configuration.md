# Zabbix 安装配置

Zabbix 是一种分布式服务监控系统，核心系统由 C语言编写。主要有服务器端 (Server)，代理服务器端 (Proxy)，被监控端 (Agent) 组成。

## Zabbix工作模式

- 主动模式

Zabbix 在主动模式下，被监控端(Agent) 会主动连接服务器端(Server)，拉取监控列表项，然后将采集到的监控数据发送到 Zabbix 的服务器端(Server)

- 被动模式

Zabbix 在被动模式下，被监控端(Agent) 会在本机监听一个端口等待服务器端(Server) 来连接拉取监控数据，Zabbix 的服务器端(Server) 会主动连接被监控端(Agent) 拉取监控数据

- 代理模式

在代理模式下，被监控端(Agent) 主动连接代理服务器端(Proxy)，将监控数据发送到代理服务器(Proxy)上，再由代理服务器(Proxy)转发到 Zabbix 的服务器端(Server) 上，这种模式通常应用在被监控端(Agent) 无法直接与服务器端(Server) 通信的情况下

## Zabbix 安装配置
  
- CentOS 7.2
- MySQL 5.5
- Zabbix 3.2.0
    
**安装 zabbix 的所需依赖包**
  
```
php php-gd php-bcmath php-ctype php-libXML php-xmlreader php-xmlwriter php-session php-net-socket
php-mbstring php-gettext php-ldap php-mysql OpenIPMI-libs OpenIPMI-modalias fping iksemel
libtool-ltdl net-snmp-libs traceroute unixODBC mariadb-libs libssh2 libcurl libcurl-devel
libxml2 net-snmp net-snmp-devel
```
  
**下载 zabbix 源码包并编译**

```
$ tar -xzvf zabbix-3.2.0.tar.gz
$ cd zabbix-3.2.0
```
  
**Zabbix Server 服务端安装**
  
```
$ ./configure --prefix=/usr/local/zabbix --enable-server --with-mysql --with-libcurl --with-libxml2 --with-net-snmp
```

开始 zabbix 的编译
  
```
$ make
$ make install
```
  
添加 zabbix 运行用户*
  
```
$ useradd -M -s /usr/bin/nologin -c "zabbix service user" zabbix
```
  
修改 zabbix Server 服务端配置文件 zabbix_server.conf
  
```
ListenIP=0.0.0.0
LogFile=/tmp/zabbix_server.log     # zabbix server 日志文件
PidFile=/tmp/zabbix_server.pid     # zabbix server 进程PID文件
DBHost=127.0.0.1                   # zabbix server 数据库地址
DBPort=3306                        # zabbix server 数据库端口
DBName=zabbix                      # zabbix server 数据库名称
DBUser=root                        # zabbix server 数据库用户
DBPassword=123456                  # zabbix server 数据库用户密码
Timeout=3
FpingLocation=/usr/bin/fping
LogSlowQueries=3000
```
  
创建 zabbix 数据库
  
```
mysql> create database zabbix;
```
  
将数据导入到 zabbix 数据库中
  
```
$ mysql -u root -p zabbix < database/mysql/schema.sql
$ mysql -u root -p zabbix < database/mysql/images.sql
$ mysql -u root -p zabbix < database/mysql/data.sql
```

zabbix web 安装
  
```
$ mkdir -p /var/www/zabbix
$ cp -R frontends/php /var/www/zabbix
$ chown -R apache:apache /var/www/zabbix
```
  
> zabbix web 默认账号： Admin 密码： zabbix
  
启动 server 服务端 (zabbix 只能以非 root 用户才能启动)
  
```
$ /usr/local/zabbix/sbin/zabbix_server -c /usr/local/zabbix/etc/zabbix_server.conf
```
  
**Zabbix Proxy 代理端安装**

安装 zabbix 所需依赖包

```
OpenIPMI-libs OpenIPMI-modalias fping iksemel libtool-ltdl net-snmp-libs traceroute
unixODBC mariadb-libs libssh2 libcurl libcurl-devel libxml2 net-snmp net-snmp-devel
```

编译 zabbix proxy 代理端
  
```
$ tar -xzvf zabbix-3.2.0.tar.gz
$ cd zabbix-3.2.0
$ ./configure --prefix=/usr/local/zabbix --enable-proxy --with-mysql --with-libcurl --with-libxml2 --with-net-snmp
$ make
$ make install
```
  
修改 zabbix_proxy.conf 配置文件 zabbix_proxy.conf

```
ProxyMode=0
Server=192.168.1.200          # zabbix server 服务器端地址
ServerPort=10051              # zabbix server 服务器端口
Hostname=zabbix proxy         # zabbix proxy 代理端主机名
LogFile=/tmp/zabbix_proxy.log # zabbix proxy 日志文件
PidFile=/tmp/zabbix_proxy.pid # zabbix proxy 进程 PID 文件
DBHost=127.0.0.1              # zabbix proxy 数据库主机地址
DBName=zabbix                 # zabbix proxy 数据库名称
DBUser=root                   # zabbix proxy 数据库用户名
DBPassword=123456             # zabbix proxy 数据库用户密码
ProxyOfflineBuffer=2          # zabbix proxy 离线数据保存 1 小时
ConfigFrequency=300           # zabbix proxy 与 server 端配置同步时间间隔
DataSenderFrequency=60
BufferSize=10240
Timeout=3
```

创建 zabbix proxy 数据库

```
mysql> create database zabbix;
```

导入 zabbix proxy 数据到数据库

```
$ mysql -u root -p zabbix < database/mysql/schema.sql
```

启动 zabbix proxy 代理服务器

```
$ /usr/local/zabbix/sbin/zabbix_proxy -c /usr/local/zabbix/etc/zabbix_proxy.conf
```
**Zabbix Agent 安装配置**
  
zabbix agent 客户端编译
  
```
$ tar -xzvf zabbix-3.2.0.tar.gz
$ cd zabbix-3.2.0
$ ./configure --prefix=/usr/local/zabbix --enable-agent
$ make
$ make install
```
  
客户端 zabbix_agentd.conf 配置文件 zabbix_agentd.conf

```
LogFile=/tmp/zabbix_agentd.log       # zabbix agentd 日志文件路径
Server=192.168.1.120                 # zabbix server 服务器 IP 地址
StartAgents=0                        # 工作模式 0 表示主动模式，1 表示被动模式
ServerActive=192.168.1.120:10051     # zabbix server 服务器 IP 地址
Hostname=CentOS                      # zabbix agentd 客户端主机名称
RefreshActiveChecks=60
BufferSize=10240
Timeout=3
```

启动 agent 客户端
  
```
$ /usr/local/zabbix/sbin/zabbix_agentd -c /usr/local/zabbix/etc/zabbix_agentd.conf
```

----------------------------------------------------------------------------------------

By typefo typefo@qq.com Update: 2017-04-29 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)
