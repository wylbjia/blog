# RabbitMQ 与 HAProxy 负载均衡

**RabbitMQ 集群节点**

- node1 192.168.1.101   磁盘节点
- node2 192.168.1.102   内存节点
- node3 192.168.1.103   内存节点
  
## 安装配置 HAProxy

```
$ yum install -y haproxy
```

HAProxy 配置文件 /etc/haproxy/haproxy.cfg

```
listen rabbitmq_cluster 0.0.0.0:5672
mode tcp
balance roundrobin
server node1 192.168.188.151:5672 check inter 3000 rise 2 fall 3
server node2 192.168.188.152:5672 check inter 3000 rise 2 fall 3
server node3 192.168.188.153:5672 check inter 3000 rise 2 fall 3
  
listen status 0.0.0.0:8000
mode http
option httplog
stats enable
stats uri /status
stats refresh 3s
```

----------------------------------------------------------------------------------------

By typefo typefo@qq.com Update: 2017-04-29 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)
