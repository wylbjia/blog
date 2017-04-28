# Nginx 编译安装

Nginx 是一个性能出色的 http 服务器和反向代理服务器

## 系统环境

- CentOS 7.3
- Nginx 1.12.0
- 开启 http2 协议
- 禁用 pop3、imap、smtp

我们只将 Nginx 用于纯 Web 服务环境，所以去掉了 pop3、imap、smtp 等邮件模块

## 安装依赖包

首先安装 nginx 的一些编译工具和依赖开发包

```
$ yum install -y gcc gcc-c++ make pcre-devel openssl-devel
```

## 编译与安装

从 Nginx 的官方网站下载源码包并解压

```
$ wget http://nginx.org/download/nginx-1.12.0.tar.gz
$ tar -xzvf nginx-1.12.0.tar.gz
```

进入 Nginx 源码包目录，然后开始编译

```
$ cd nginx-1.12.0
$ ./configure --prefix=/usr/local/nginx --with-threads --with-file-aio --with-http_ssl_module
              --with-http_v2_module
              --without-mail_pop3_module
              --without-mail_imap_module
              --without-mail_smtp_module
```

> 如果在执行 configure 环境检测过程中报错，一般情况下可能是缺少或者没有安装相关的依赖包

开始 Nginx 编译和安装

```
$ make
$ make install
```

手动启动 nginx 服务

```
$ /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

## systemd 服务脚本

将以下内容保存为 nginx.service 文件，然后复制到 /etc/systemd/system 目录下

```
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network.target

[Service]
Type=forking
PIDFile=/run/nginx.pid

ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

开启 nginx.service 服务开机自启动

```
$ systemctl enable nginx.service
```

启动 nginx.service 服务

```
$ systemctl start nginx.service
```

-----------------------------------

By typefo <typefo@qq.com> Update: 2017-04-27 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)
